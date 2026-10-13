# Lumissary — *The Master and His Emitter*

Two 3D-printed brain hemispheres that light up from inside, and that sense each other.

Each hemisphere is filled with addressable LEDs and has an ESP32 with WiFi and UWB radio. The pair measure the live distance between them and use it to sync their light. Hold them close together and their pulses lock into one rhythm. Move them apart and they drift, each falling back into a tempo of its own.

The two halves don't light up the same way. One fires in sharp, separate points of light. The other glows in soft, blended washes. They share a rhythm when they're near each other, but each expresses it in its own way.

This document covers the concept and the full technical design as of the end of the build-planning phase.

---

## 1. The Idea

This project comes straight from **Iain McGilchrist's *The Master and His Emissary***. While I was reading it, the book's argument about the two hemispheres of the brain kept turning into a physical image of this build.

McGilchrist describes the left hemisphere as narrow and analytical, breaking the world into details. He describes the right as broad and integrative, seeing things whole and in relation. Lumissary builds that difference into both the hardware and the behavior of each hemisphere:

| | Left hemisphere | Right hemisphere |
|---|---|---|
| **Character** | Analytical, focused on parts | Integrative, focused on relations |
| **Construction** | Bare LED strips: each LED stays a distinct, separate point | Diffused light: a clear tube or sleeve blends neighboring LEDs together *(design intent, not yet sourced)* |
| **Firing pattern** | Rigid, step by step: one LED or one color at a time | Smooth and continuous: colors blend from one state to the next |

What connects them is a **coupled-oscillator model** (Kuramoto) driven by physical distance. The synchronization is the point of the piece, not a technical add-on: it turns the relationship between the two hemispheres, separate or unified, into something you can see. That tension is what the book is about.

---

## 2. What You See

- **Hemispheres held close together:** both lock to the same tempo. The left steps through sharp, discrete beats and the right flows through smooth, blended ones, on the same beat.
- **Hemispheres pulled apart:** the shared tempo loosens. Each one drifts toward its own natural frequency, and the phase offset between them visibly grows. The left's steps and the right's flows fall out of step.
- **Somewhere in between:** the coupling weakens gradually with distance, so the pair pass through a range of partial sync. They are never fully locked and never fully free.

Distance controls *how tightly* the two rhythms are locked. Each hemisphere's own *character* controls how it expresses that rhythm.

---

## 3. How the Sync Works (Kuramoto Model)

This model is the behavioral core of the pair, and the reason the build includes a UWB radio at all.

Each hemisphere's firing pattern is an oscillator with its own phase θ and natural frequency ω. The two are coupled:

```
dθ₁/dt = ω₁ + K(d)·sin(θ₂ − θ₁)
dθ₂/dt = ω₂ + K(d)·sin(θ₁ − θ₂)
```

**K(d) is the coupling strength, and it depends on the measured distance `d` between the hemispheres.** This term is what links the visuals to physical proximity.

- **UWB provides `d`:** the DW1000 radios range between the two ESP32 modules and update the distance continuously.
- **WiFi carries phase:** each hemisphere broadcasts its current θ to the other at a regular interval, using low-latency UDP or ESP-NOW-style messaging.
- **Coupling falls off with distance:** `K(d)` is high when the hemispheres are close (phases pull together quickly) and falls as they move apart (phases drift). Either `K(d) = K₀ / d` or `K₀·e^(−d/λ)` is a reasonable starting point. The exact curve is a tuning parameter, not a fixed formula.

**Rendering:** both hemispheres use the *same* θ relationship, and only the rendering differs. The left maps phase to a rigid step function. The right maps it to a continuous, interpolated blend, and the diffusion tubing adds physical softening on top.

**Firmware notes:**
- Everything runs in ESP32 firmware. No hardware is needed beyond what's listed below.
- ω₁ and ω₂ can be equal, so sync depends purely on distance. They can also differ slightly, which gives each hemisphere a baseline "personality" and lets distance control how much of it shows through.
- The UWB ranging rate and the WiFi broadcast rate both need to be faster than the visible pulse cycle, or the sync won't look smooth.
- Prototype `K(d)` in software first: simulate two oscillators with a fake distance input before connecting live UWB data.
- Keep the sync logic separate from rendering: two rendering functions, both reading the same θ.

---

## 4. Hardware

### At a Glance

| Spec | Value |
|---|---|
| LED strips | 3× Adafruit DotStar (SK9822 driver), 144 LED/m, cut to ~7in each (~26 LEDs/strip) |
| Controller | ESP32 UWB module (Makerfabs): WiFi + DW1000 UWB radio |
| Level shifter | 1× SN74HCT245 (3.3V logic in → 5V logic out) |
| Board | Single consolidated protoboard, double-sided/plated-through-hole, 12x18cm |
| Power | 2S LiPo (7.4V), split into two independently regulated 5V rails |
| Runtime target | ~2 hours at 80%-capped brightness |

### Power Architecture

The design uses **two electrically separate 5V rails that share one battery and are never tied together:**

- **LED bus:** Battery → XT30 splitter → 5A fuse → FEICHAO 8A UBEC → main +5V bus → 3× JST connectors → strips
- **ESP32 rail:** Battery → XT30 splitter → 2A fuse → Yuenhoang 3A UBEC (isolated) → ESP32 5V-in

**Why two rails:** switching regulators produce ripple, and LED current spikes whenever brightness or patterns change. A shared rail would couple that noise directly into the WiFi/UWB radio's supply. The ranging that drives the whole piece depends on that radio. Separate rails with separate fuses mean a fault on one side can't affect the other.

**Sizing:**
- 3 strips × 26 LEDs × 60mA × 0.8 brightness cap ≈ **3.75A worst case** (LED bus)
- ESP32 WiFi/UWB transmit bursts ≈ **1-2A** (isolated rail)
- Realistic average (30% duty cycle) ≈ 6.6W total → **~17-18Wh battery target** → 2000mAh 2S pack, ~1h40-2h runtime

### Circuit Design

- **Pin order P-G-D-C** (not flanking or interleaved): power and ground sit next to each other, so Data and Clock run straight from JST to chip to ESP32 with no forced wire crossings.
- **Front/back split:** the whole +5V network (P pins, chip VCC, ESP32 power) is on the front face. Everything else (GND, Data, Clock, chip body, ESP32 body) is on the back. This removes the need for insulated hop wires.
- **Near-edge / far-edge placement:** the JSTs, level shifter, and power circuitry sit at one edge. The ESP32 and its antenna sit at the opposite edge, which gives as much RF separation as the board allows.
- **220Ω series resistors** on all 6 Data/Clock lines, on the level-shifter *output* side. They damp high-frequency harmonics from fast logic edges, reducing interference with the nearby UWB receiver.
- **UBEC/BEC modules are external and inline,** spliced into the wiring like the fuse holder rather than soldered to the board.

### RF / Antenna

- **DW1000 keep-out:** at least 10mm with no metal on either side of the antenna (per the Qorvo/Decawave datasheet). Check this against the actual Makerfabs antenna position before fixing enclosure dimensions.
- **Board length isn't used as an RF spacer.** The gap is kept close to the minimum needed for solder access, trading some margin for a smaller board. Add shielding if testing shows interference.
- **Mitigations:** resistor-damped signal lines, the isolated ESP32 rail, physical zone separation on the board, and two regulators instead of one large converter.
- **Still to verify:** UWB ranging accuracy and WiFi signal quality at the real build spacing.

### Enclosure & Thermal

- **Shape:** a 3D-printed hemisphere in the form of a brain, with the strips lining the inside, moderately packed.
- **Material:** use PETG or ABS rather than PLA. The LED contact points get warm enough to soften PLA.
- **Venting:** exhaust at the crown and intake at the base, using the dome's natural convection. Add a small secondary vent at the very top if the main vent isn't there.
- **Fan:** not required at the 3-strip scale (~8.6W realistic average, ~25W worst case). Revisit if the enclosure ends up smaller or more sealed.
- **Strip mounting:** leave a small air gap between the strips and the printed wall so air can reach the back of the strip.
- **Right-hemisphere diffusion tubing (not yet sourced):** a tight sleeve could trap heat against the strip even with a good wall gap. The tube's fit and ventilation need checking once a material is chosen.

---

## 5. Building It

### Build Phases

1. **Fixed anchors:** ESP32 stacking headers, JST connector housings
2. **Back-face signal skeleton:** GND bus, JST-to-chip wires (with 220Ω resistors), chip-to-ESP32 wires. Check continuity before the chip goes in.
3. **Seat the level shifter:** chip, GND, OE/DIR straps, decoupling cap. Bench-power test before going further.
4. **Front-face +5V network:** P-pin taps, chip VCC via, ESP32 5V-in via
5. **Power section:** XT30 inlet, splitter, two fused branches, both external UBEC modules
6. **Integration test:** multimeter-check both rails before connecting anything. Connect the battery last, and test one strip before committing all three.

### Shopping List

See `final_shopping_list.md` for the complete, verified-in-stock parts list. It is organized by category (board/core, power, connectors, wire, capacitors) and includes purchase links.

**The list covers one hemisphere.** For the second, buy another battery, both UBECs, the ESP32 module, and the LED strips. Most connector, wire, and capacitor packs have enough spare to cover a second build.

---

## 6. Status & Open Items

**Hardware**
- [ ] Confirm the exact antenna location and keep-out dimension on the physical Makerfabs board
- [ ] Confirm the ESP32 board has a usable 5V/VIN input pin (not just USB power)
- [ ] Verify the wire gauge fits in the XT30 solder cups (18AWG trunk wire may be snug)
- [ ] Confirm the strip wire-to-pin mapping (which color carries which signal) once the strips arrive, before final soldering

**Sync & firmware**
- [ ] Prototype the Kuramoto coupling function `K(d)` in software before connecting live UWB data
- [ ] Test UWB ranging accuracy and WiFi signal at actual enclosure spacing
- [ ] Tune the UWB ranging rate and WiFi phase-broadcast rate against the visible LED pulse cycle

**Creative**
- [ ] Source and choose a material for the right hemisphere's diffusion tubing (design intent only, not yet on the shopping list)
