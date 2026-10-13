# Lumissary — *The Master and His Emitter*

A 3D-printed hemisphere of a human brain, interior-lit by addressable LED strips, with onboard WiFi + UWB positioning via an ESP32 module. Two hemispheres are built as a pair: they communicate with each other over WiFi and use UWB ranging to measure the live distance between them, driving a Kuramoto-style coupled-oscillator model that synchronizes their LED firing patterns — the closer the two hemispheres are to each other, the more tightly their pulses lock in phase; the farther apart, the more phase drift appears between them. This document summarizes the final design as of the end of the build-planning phase.

---

## 1. Overview

| Spec | Value |
|---|---|
| LED strips | 3× Adafruit DotStar (SK9822 driver), 144 LED/m, cut to ~7in each (~26 LEDs/strip) |
| Controller | ESP32 UWB module (Makerfabs) — WiFi + DW1000 UWB radio |
| Level shifter | 1× SN74HCT245 (3.3V logic in → 5V logic out) |
| Board | Single consolidated protoboard, double-sided/plated-through-hole, 12x18cm |
| Power | 2S LiPo (7.4V), split into two independently-regulated 5V rails |
| Runtime target | ~2 hours at 80%-capped brightness |

---

## 2. Design Philosophy & Inspiration

This project is directly inspired by **Iain McGilchrist's *The Master and His Emissary*** — while reading it, the two-hemisphere brain-lateralization argument kept resolving into a physical image of exactly this build. McGilchrist's core distinction (the left hemisphere as narrow, analytical, detail-atomizing; the right as broad, integrative, relational) maps onto both the **physical construction** and the **firing behavior** of the two hemispheres:

- **Left hemisphere — bare LED strips.** No diffusion, individual LEDs stay visually distinct and separable — light as discrete, analyzable units.
- **Right hemisphere — diffused, blended light** *(design intent, not yet sourced)*: a clear tube or sleeve around the strip so adjacent LEDs' light bleeds together, colors blending rather than staying separable. This is a real intended part of the build, just not on the shopping list yet — material/sourcing to be decided later.
- **Firing pattern reflects the same split** — see [Section 6](#6-inter-hemisphere-synchronization-kuramoto-model): the left renders in rigid, atomic, one-color-at-a-time steps; the right renders the same underlying shared rhythm as smooth, continuous blends.

The Kuramoto synchronization model isn't just a technical add-on, then — it's the mechanism that lets the two hemispheres' relationship (independent vs. unified, depending on distance) stay legible as physical, visual behavior, which is the same tension the book is actually about.

---

## 3. Power Architecture

**Two electrically separate 5V rails, one shared battery, never tied together:**

- **LED bus**: Battery → XT30 splitter → 5A fuse → FEICHAO 8A UBEC → main +5V bus → 3× JST connectors → strips
- **ESP32 rail**: Battery → XT30 splitter → 2A fuse → Yuenhoang 3A UBEC (isolated) → ESP32 5V-in

**Why two rails instead of one:** switching regulators produce ripple, and LED current draw creates real transients as brightness/patterns change. Sharing one rail between the LEDs and the ESP32 would let that noise couple directly into the WiFi/UWB radio's supply. Keeping them separate — with independent fusing per branch — means a fault or current spike on one side can't affect the other, and neither regulator has to fight the other's output.

**Numbers behind the sizing:**
- 3 strips × 26 LEDs × 60mA × 0.8 brightness cap ≈ **3.75A worst case** (LED bus)
- ESP32 WiFi/UWB transmit bursts ≈ **1-2A** (isolated rail)
- Realistic average (30% duty cycle) ≈ 6.6W total → **~17-18Wh battery target** → 2000mAh 2S pack, ~1h40-2h runtime depending on actual usage

---

## 4. Circuit Design — Key Decisions

- **Pin order: P-G-D-C** (not flanking/interleaved) — keeps power and ground pins adjacent, letting Data/Clock drop straight from JST to chip to ESP32 with zero forced wire crossings.
- **Front/back split**: the entire +5V network (P pins, chip VCC, ESP32 power) lives on the board's front face; everything else (GND, Data, Clock, chip body, ESP32 body) lives on the back. This eliminates the need for insulated "hop" wires arching over other traces.
- **Near-edge / far-edge placement**: JSTs, level shifter, and all power circuitry cluster at one edge of the board; the ESP32 (and its antenna) sit at the opposite edge, maximizing the one dimension of RF separation the board layout can control.
- **220Ω series resistors** on all 6 Data/Clock lines, placed on the level-shifter *output* side (not the ESP32 input side) — damps high-frequency harmonic content from the fast logic edges, reducing potential interference with the nearby UWB receiver.
- **UBEC/BEC are external inline modules**, not board-mounted components — same category as the fuse holder, spliced into the wire rather than soldered onto board copper.

---

## 5. RF / Antenna Considerations

- **DW1000 keep-out spec**: minimum 10mm, no metal, on either side of the antenna (per Qorvo/Decawave datasheet) — verify against the actual Makerfabs board's antenna placement before finalizing enclosure dimensions.
- **Board length is not used as a dedicated RF spacer** — the near/far-edge layout keeps the gap close to the minimum needed for solder access. This trades some layout-level margin for a smaller board; compensate with shielding if testing shows interference.
- **Mitigations in place**: resistor-damped signal lines, isolated ESP32 power rail, physical zone separation on the board, dual-regulator design (reduces total noise vs. a single large converter).
- **Still to verify**: actual UWB ranging accuracy and WiFi signal quality at real build spacing — test before final assembly/gluing.

---

## 6. Inter-Hemisphere Synchronization (Kuramoto Model)

This is the behavioral core of the two-hemisphere pair — it's what the UWB radio is actually *for* in this project, beyond just being an interesting module to include.

**Concept**: each hemisphere's LED "firing" pattern is treated as a coupled oscillator with its own phase θ and natural frequency ω. Two coupled oscillators following the Kuramoto model evolve as:

```
dθ₁/dt = ω₁ + K(d)·sin(θ₂ − θ₁)
dθ₂/dt = ω₂ + K(d)·sin(θ₁ − θ₂)
```

where **K(d) is the coupling strength, a function of the measured distance `d` between the two hemispheres** — this is the whole mechanism that ties the visual behavior to physical proximity.

**How distance drives the effect:**
- **UWB provides `d`** — the DW1000 radios perform ranging between the two ESP32 modules, giving a live distance measurement, refreshed continuously.
- **WiFi carries the phase-sync channel** — each hemisphere broadcasts its current phase θ to the other over WiFi (UDP/ESP-NOW-style low-latency messaging) at a regular interval.
- **Coupling strength scales inversely with distance**: `K(d)` is high when `d` is small (strong coupling → phases pull toward each other quickly) and falls off as `d` increases (weak coupling → phases drift apart). An inverse or exponential falloff (e.g. `K(d) = K₀ / d` or `K₀·e^(−d/λ)`) are both reasonable starting points — the exact curve is a tuning parameter, not a fixed formula.

**Rendering layer — same shared phase, rendered differently per hemisphere:** both hemispheres are driven by the *same* underlying θ relationship from the equations above — the sync/drift behavior itself doesn't change between them. What differs is how each hemisphere expresses that shared phase visually, matching the analytic/holistic split from Section 2:
- **Left hemisphere**: renders the phase as a rigid, atomic step function — one LED (or one color) fires at a time, discrete transitions, no blending between states.
- **Right hemisphere**: renders the same phase as a continuous, smoothly interpolated blend — helped physically by the diffusion tubing, which blurs adjacent LEDs together rather than keeping them visually separable.

So proximity still governs how *tightly locked* the two hemispheres' rhythms are, exactly as before — it's the *character* of each hemisphere's response to that shared rhythm that now differs, one rigid and discrete, one fluid and blended.

**What this looks like in practice:**
- Hemispheres held close together → both rhythms lock together — the left stepping through sharp, discrete beats, the right flowing through smooth blended ones, both on the same tempo
- Hemispheres moved apart → the shared tempo loosens, each hemisphere drifts at its own natural frequency, with a visibly growing phase offset between them — the left's steps and the right's flows fall out of step with each other

**Implementation notes / open questions:**
- Lives entirely in ESP32 firmware — no additional hardware beyond what's already in this README (UWB for ranging, WiFi for the phase-sync channel).
- Natural frequencies (ω₁, ω₂) can be set equal (pure distance-driven sync) or slightly different (adds a baseline "personality" per hemisphere, with distance modulating how much that personality shows through).
- Update rate for both the UWB ranging and the WiFi phase broadcast needs tuning against the LED refresh rate — the sync effect only reads as smooth if the underlying phase updates faster than the visible pulse cycle.
- Coupling function `K(d)` and its falloff rate are the main creative/tuning parameters — worth prototyping in software (simulate two oscillators with a fake distance input) before wiring it to live UWB data.
- The left hemisphere's discrete-step rendering and the right's blended rendering both need to be built as separate rendering functions consuming the *same* θ value — worth architecting the firmware so the sync logic and the rendering logic are cleanly separated from the start.

---

## 7. Enclosure & Thermal

- **Shape**: 3D-printed hemisphere (brain form), strips lining the interior, moderately packed.
- **Material**: PETG or ABS recommended over PLA — LED contact points run warm enough to risk softening PLA.
- **Venting**: crown (top) exhaust + base (low) intake, using the dome's natural convection — hot air rises and exits at the crown, pulling cool air in at the base. A small secondary vent at the true crown is worth adding if the main vent isn't at the very top.
- **Fan**: not required at current 3-strip scale (~8.6W realistic average, ~25W worst case) — passive venting should cover it. Revisit if the enclosure ends up smaller/more sealed than planned.
- **Strip mounting**: leave a small air gap between strips and the printed wall rather than flush contact, so airflow can actually reach the back of the strip.
- **Right hemisphere diffusion tubing (design intent, not yet sourced)**: worth revisiting this air-gap guidance once the tube material is chosen — a sleeve wrapped tightly around the strip could trap heat against it even if the strip-to-wall gap is fine, so the tube's own fit/ventilation matters independently.

---

## 8. Build Phases

1. **Fixed anchors** — ESP32 stacking headers, JST connector housings
2. **Back-face signal skeleton** — GND bus, JST-to-chip wires (with 220Ω resistors), chip-to-ESP32 wires; continuity-check before the chip goes in
3. **Seat the level shifter** — chip, GND, OE/DIR straps, decoupling cap; bench-power test before proceeding
4. **Front-face +5V network** — P-pin taps, chip VCC via, ESP32 5V-in via
5. **Power section** — XT30 inlet, splitter, two fused branches, both external UBEC modules
6. **Integration test** — multimeter-check both rails before connecting anything; battery connected last; test one strip before committing all three

---

## 9. Shopping List

See `final_shopping_list.md` for the complete, verified-in-stock parts list, organized by category (board/core, power, connectors, wire, capacitors) with direct purchase links.

**Note**: this list covers **one hemisphere**. For a second hemisphere, the battery, both UBECs, the ESP32 module, and the LED strips need fresh purchases (sold as single units or with pack sizes that don't stretch across two builds) — most connector/wire/capacitor packs already have enough margin to cover a second build without additional purchases.

---

## 10. Open Items / Things to Verify Before Final Assembly

- [ ] Confirm exact antenna location and keep-out dimension on the physical Makerfabs board
- [ ] Confirm ESP32 board has a usable 5V/VIN input pin (not just USB-only power)
- [ ] Test UWB ranging accuracy and WiFi signal at actual enclosure spacing
- [ ] Verify wire gauge fits comfortably in XT30 solder cups (18AWG trunk wire may be snug)
- [ ] Confirm strip wire-to-pin mapping (color-to-signal) once strips are in hand, before final soldering
- [ ] Prototype the Kuramoto coupling function `K(d)` in software before wiring to live UWB data
- [ ] Tune UWB ranging refresh rate and WiFi phase-broadcast rate against the visible LED pulse cycle
- [ ] Source and choose material for the right hemisphere's diffusion tubing (design intent only — not yet on the shopping list)
