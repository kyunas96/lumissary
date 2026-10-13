# LED Brain Hemisphere — Phased Build & Solder Guide

> **Cross-check results:** the battery and protoboard links here are duplicate Amazon listings of the identical product as `final_shopping_list.md`'s links — either works. The fuse holder link below (VicVan, 12AWG) is genuinely better-spec than the list's original pick and has been adopted there too. The SN74HCT245 link below is the **single-piece** listing — `final_shopping_list.md` uses the **5-pack** instead, for spares; pick whichever quantity you actually want, both are the authentic Juried Engineering part.

Ordered so each phase ends in a **test gate**: a quick check with a multimeter (and later, power) before you commit the next set of parts. The goal is to never solder a hard-to-reach component on top of an untested one.

**A note on buying:** almost everything here comes in multi-packs, so you'll place one order up front. The per-phase lists below tell you which parts get *consumed* at each stage so you can stage them on the bench in order — not that you buy in six separate trips. Amazon links (US availability) are collected in the [Consolidated shopping list](#consolidated-shopping-list-one-order-us-amazon-links) at the bottom, and each phase table links the specific line item.

**General solder rules (apply every phase):**
- Front face = +5V-family nets only; back face = GND + signal + component bodies. Through-holes (vias) carry a net from one face to the other.
- Solder low/heat-tolerant parts before tall/heat-sensitive ones. Resistors, wire jumpers, and straps first; connectors next; tall headers and the ESP module last.
- Slide heat-shrink onto wire *before* soldering. Iron ~350–380 °C. Pre-tin wire and pre-fill XT30 cups separately, then join, to limit heat soak into plastic.

---

## Phase 0 — Tools & consumables (have before you start)
Not part of the design, but you can't build without them. You already own the soldering iron, solder, and multimeter, so those are omitted from the buy list below — but note the multimeter is still what makes every test gate possible, so keep it on the bench.

- Wire strippers, flush cutters, tweezers, helping-hands / vise
- Heat-shrink tubing assortment (the Frienda XT30 kit includes some)
- **Recommended add-on not on your original list:** a **20-pin DIP socket** for the SN74HCT245 (~$1 for a pack). Solder the socket, drop the chip in after everything is tested — protects the chip from iron heat and lets you swap it if a data line is ever damaged. → [Ganasome 20-pin DIP socket, 20-pack](https://www.amazon.com/Ganasome-Spacing-Machine-Through-Hole-20-Pack/dp/B0CX8XM2YW)

**Test gate:** iron heats and tins cleanly; multimeter reads continuity and 5 V DC.

---

## Phase 1 — Off-board power harness (build & test *before* touching the board)
**Why first:** if a regulator is miswired or set to the wrong voltage, you want to find out on the bench, not after it's feeding your board. This phase produces two known-good 5 V pigtails.

**Solder order:** battery pigtail → splitter adapters → each regulator's input; then fit XT30 plugs on each regulator's *output* so they can mate to the board later.

**Components consumed:**

| Qty | Component | Amazon link (US) |
|---|---|---|
| 1 | 2S LiPo battery, XT30 (Fytoo 2S 2000mAh) | [Fytoo 2S 7.4V 2000mAh XT30](https://us.amazon.com/Fytoo-Battery-2000mAh-Charging-Quadcopter/dp/B0FH9C5T9L) |
| 1 | XT30 fused pigtail, 15A, 14AWG | [XT30 14AWG pigtail w/ 15A fuse](https://www.amazon.com/Connector-Silicone-Pre-Fitted-Waterproof-Connectors/dp/B0GZQX6SRZ) |
| 1 | XT30 Y-splitter (1F→2M) | [XT30 1F→2M parallel splitter](https://www.amazon.com/Parallel-Connector-RC-Multicopter-Quadcopter/dp/B0D9LFV1GM) |
| 1 | 8A UBEC (LED bus regulator) | [FEICHAO 8A UBEC 5V/6V](https://www.amazon.com/FEICHAO-Output-7V-25-5V-Switch-Helicopters/dp/B07DD9L6P6) |
| 1 | 3A BEC (ESP32 regulator) | [Yuenhoang 3A BEC 5V/6V switchable](https://www.amazon.com/Yuenhoang-Adjustable-Shielding-Antijamming-Switchable/dp/B0D2NHBKJJ) |
| ~3 pr | XT30 bare solder-cup connectors (adapters/output plugs) | [Frienda 20-pair XT30 + heat-shrink](https://www.amazon.com/Connectors-Female-Pieces-Shrink-Battery/dp/B0875MBLNH/) |
| — | Heat-shrink | Frienda kit (above) |

**Test gate (do NOT skip):**
1. Set the small BEC's voltage jumper to **5 V** (it ships switchable — confirm it's not on 6 V).
2. Plug battery → pigtail → splitter → both regulators. **Multimeter on each regulator output: expect 5.0 V** (±0.2 V), correct polarity.
3. Confirm the 15 A fuse is actually seated in the pigtail.
4. Unplug the battery before moving to board work.

---

## Phase 2 — On-board power entry: ports, fuses, buses, bulk cap
**Why now:** establishes the two 5 V rails and common ground on the board so every later phase has something to power from and test against.

**Solder order:** 2× on-board XT30 ports → 5 A and 2 A inline fuse holders right after each port → run the 18 AWG **LED+5V bus** (front) and **GND bus** (back) → run the **ESP-5V lane** (front, kept clear of the LED bus) → 1000 µF bulk cap across LED+5V→GND just after the 5 A fuse.

**Components consumed:**

| Qty | Component | Amazon link (US) |
|---|---|---|
| 1 | Protoboard, double-sided PTH, 12×18 cm | [Double-sided PTH protoboard 12×18 cm (5-pack)](https://www.amazon.com/Double-Sided-Universal-Packs-12x18/dp/B08WJD6G3L) |
| 2 | XT30 bare solder-cup ports (on-board) | [Frienda 20-pair XT30](https://www.amazon.com/Connectors-Female-Pieces-Shrink-Battery/dp/B0875MBLNH/) |
| 2 | Inline fuse holders (+ blade fuses incl. 5A & 2A) | [Inline fuse holders, 5-pack w/ assorted fuses](https://www.amazon.com/VicVan-Pack-Inline-Fuse-Holder/dp/B0CLK77GYN) |
| 1 | 5 A blade fuse (LED branch) | (from fuse pack above) |
| 1 | 2 A blade fuse (ESP branch) | (from fuse pack above) |
| 1 | 1000 µF electrolytic cap, 16 V+ | [1000 µF 25 V radial electrolytic](https://www.amazon.com/1000uF-Capacitor-Electrolytic-General-purpose/dp/B00CDTA9WK) |
| — | 18 AWG stranded (buses) | [TUOFENG 18 AWG silicone stranded, 6 colors](https://www.amazon.com/TUOFENG-18-Stranded-Insulated-Automotive/dp/B07G744V5Z) |

**Test gate:**
1. Plug in the Phase-1 harness. **LED+5V bus reads 5 V** after its 5 A fuse; **ESP-5V lane reads 5 V** after its 2 A fuse.
2. **Rail isolation check (critical):** multimeter continuity between LED+5V and ESP-5V should read **open** — they must never be tied.
3. Continuity from each rail's GND return to the common GND bus = closed.
4. Bulk cap polarity correct (− stripe leg on GND). Unplug power.

---

## Phase 3 — Level shifter (SN74HCT245)
**Why now:** it's the heart of the signal path and sits centrally; place it before the connectors crowd around it.

**Solder order:** DIP-20 socket (recommended) → 0.1 µF decoupling cap right across the chip's VCC/GND → VCC via to LED+5V (front) → GND lane (back) → **OE→GND** strap (enable) → **DIR→VCC** strap (forces A→B). Insert the chip only after Phase-3 testing.

**Components consumed:**

| Qty | Component | Amazon link (US) |
|---|---|---|
| 1 | SN74HCT245N, DIP-20 | [Juried Engineering SN74HCT245N (also sold as 20-pack)](https://www.amazon.com/Juried-Engineering-SN74HCT245N-Transceivers-Breadboard-Friendly/dp/B08FZMJPPQ) |
| 1 | 20-pin DIP socket (recommended) | [Ganasome 20-pin DIP socket, 20-pack](https://www.amazon.com/Ganasome-Spacing-Machine-Through-Hole-20-Pack/dp/B0CX8XM2YW) |
| 1 | 0.1 µF ceramic cap | [BOJACK 0.1 µF 50 V ceramic, 50-pack](https://www.amazon.com/BOJACK-Capacitors-Low-Voltage-Dielectric-Capacitor/dp/B07X5XTDPB) |
| — | 22 AWG solid for straps | [TUOFENG 22 AWG (silicone stranded — swap to solid if you prefer stiff straps)](https://www.amazon.com/Electric-Flexible-Silicone-different-Electronic/dp/B07G2JWYDW) |

**Test gate:**
1. With chip socket empty, power on: **VCC pin position reads 5 V**, GND pin position reads 0 V.
2. Confirm **OE strap → GND** (continuity to GND) and **DIR strap → VCC** (continuity to LED+5V).
3. Power off, insert the SN74HCT245 into the socket (pin-1 notch correct).

---

## Phase 4 — Strip connectors + data/clock damping
**Solder order:** 3× JST-SM connectors → their **P→LED+5V** (front) and **G→GND** (back) → 6× 220 Ω resistors on the chip's **B-output** side → D/C wires from each resistor to its JST D/C pin.

**Components consumed:**

| Qty | Component | Amazon link (US) |
|---|---|---|
| 3 | JST-SM 4-pin, 20 AWG (SK9822-compatible) | [BTF-LIGHTING 4-pin JST-SM, 5-pack](https://www.amazon.com/BTF-LIGHTING-Connector-Extension-Solderless-Controller/dp/B0DCB76FP4) |
| 6 | 220 Ω resistors | already owned |
| — | 22 AWG solid (D/C runs) | [TUOFENG 22 AWG](https://www.amazon.com/Electric-Flexible-Silicone-different-Electronic/dp/B07G2JWYDW) |

**Test gate:**
1. Continuity: each JST **P → LED+5V bus**, each **G → GND bus**.
2. Each JST **D/C pin → 220 Ω → correct chip B-output** (check the D1/C1→JST-1 … D3/C3→JST-3 mapping).
3. No continuity between any adjacent D and C (no solder bridges).

---

## Phase 5 — ESP32 mount + control lines (system test)
**Why last for actives:** the ESP sits on tall stacking headers for antenna clearance and shouldn't be in the way while you solder everything else. It plugs in — you don't solder the module itself.

**Solder order:** female stacking headers (from male-pin alignment) at the ESP footprint → ESP **5V-IN** from the ESP-5V lane (front) → ESP **GND** (back) → 6× 22 AWG **A-input** lanes from chip A-side to ESP GPIO (D1 C1 D2 C2 D3 C3). Seat the ESP module on the headers.

**Components consumed:**

| Qty | Component | Amazon link (US) |
|---|---|---|
| 1 set | Female stacking headers, extra tall | [Cermant 1×40 extra-tall female stacking header, 10-pack](https://www.amazon.com/Cermant-Connector-Female-Stacking-Header%EF%BC%8CNeedle/dp/B0CJNF94HW) |
| as needed | Male breakaway header pins | [Gikfun 1×40 breakaway male header pins, 10-pack](https://www.amazon.com/Gikfun-2-54mm-Single-Breakaway-Arduino/dp/B00U8OCENY) |
| 1 | ESP32 UWB module | Makerfabs (sourced separately — not on Amazon) |
| — | 22 AWG solid (A-input lanes) | [TUOFENG 22 AWG](https://www.amazon.com/Electric-Flexible-Silicone-different-Electronic/dp/B07G2JWYDW) |
| 1 set | DotStar/SK9822 strips (for the live test) | [Adafruit direct](https://www.adafruit.com/product/2242) or [BTF-LIGHTING SK9822 strip (Amazon)](https://www.amazon.com/BTF-LIGHTING-Individually-Addressable-Non-Waterproof-Separately/dp/B07BPX2KFD) |

**Test gate:**
1. Before seating the ESP: **5V-IN position reads 5 V**, GND reads 0 V.
2. Seat ESP, power up — it should boot (LED/serial).
3. Load test firmware, plug one strip into JST-1, confirm it lights and addresses correctly; repeat for JST-2/-3.
4. **Measure ESP32 peak current here** (inline meter on the ESP-5V lane during WiFi/UWB) — if peak is ≤ ~0.7 A, swap the 2 A fuse for a **1 A** per the audit.

---

## Phase 6 — Fan port + finish
**Solder order:** 2-pin JST fan port → **P→LED+5V** (front), **G→GND** (back). Populate the connector even if you won't run a fan yet.

**Components consumed:**

| Qty | Component | Amazon link (US) |
|---|---|---|
| 1 | 2-pin JST connector, M+F | [2-pin JST-SM male+female pigtail, 10-pair](https://www.amazon.com/Pairs-Connector-Female-Cable-Pigtail/dp/B0F2MDYPXM) |
| (opt) | 30 mm 5 V fan | not on list — optional, add later |

**Final test gate:** full smoke test — all three strips + ESP running together off the battery for several minutes; check regulators and the SN74HCT245 aren't getting hot; re-confirm LED+5V and ESP-5V stay isolated under load.

---

## Consolidated shopping list (one order, US Amazon links)
Everything you need to buy in a single cart, grouped by the phase that consumes it. Soldering iron, solder, and multimeter are intentionally omitted — you already own them.

| Component | Phase | Amazon link (US) |
|---|---|---|
| Fytoo 2S 7.4V 2000mAh LiPo, XT30 | 1 | [link](https://us.amazon.com/Fytoo-Battery-2000mAh-Charging-Quadcopter/dp/B0FH9C5T9L) |
| XT30 fused pigtail, 15A, 14AWG | 1 | [link](https://www.amazon.com/Connector-Silicone-Pre-Fitted-Waterproof-Connectors/dp/B0GZQX6SRZ) |
| XT30 Y-splitter (1F→2M) | 1 | [link](https://www.amazon.com/Parallel-Connector-RC-Multicopter-Quadcopter/dp/B0D9LFV1GM) |
| FEICHAO 8A UBEC (LED bus reg.) | 1 | [link](https://www.amazon.com/FEICHAO-Output-7V-25-5V-Switch-Helicopters/dp/B07DD9L6P6) |
| Yuenhoang 3A BEC 5V/6V (ESP reg.) | 1 | [link](https://www.amazon.com/Yuenhoang-Adjustable-Shielding-Antijamming-Switchable/dp/B0D2NHBKJJ) |
| Frienda 20-pair XT30 connectors + heat-shrink | 1–2 | [link](https://www.amazon.com/Connectors-Female-Pieces-Shrink-Battery/dp/B0875MBLNH/) |
| Double-sided PTH protoboard, 12×18 cm (5-pack) | 2 | [link](https://www.amazon.com/Double-Sided-Universal-Packs-12x18/dp/B08WJD6G3L) |
| Inline fuse holders, 5-pack w/ assorted blade fuses (5A, 2A, 1A) | 2 | [link](https://www.amazon.com/VicVan-Pack-Inline-Fuse-Holder/dp/B0CLK77GYN) |
| 1000 µF 25 V radial electrolytic cap | 2 | [link](https://www.amazon.com/1000uF-Capacitor-Electrolytic-General-purpose/dp/B00CDTA9WK) |
| TUOFENG 18 AWG silicone stranded wire | 2 | [link](https://www.amazon.com/TUOFENG-18-Stranded-Insulated-Automotive/dp/B07G744V5Z) |
| SN74HCT245N, DIP-20 (Juried Engineering) | 3 | [link](https://www.amazon.com/Juried-Engineering-SN74HCT245N-Transceivers-Breadboard-Friendly/dp/B08FZMJPPQ) |
| 20-pin DIP socket (Ganasome 20-pack) | 0/3 | [link](https://www.amazon.com/Ganasome-Spacing-Machine-Through-Hole-20-Pack/dp/B0CX8XM2YW) |
| 0.1 µF 50 V ceramic cap (BOJACK 50-pack) | 3 | [link](https://www.amazon.com/BOJACK-Capacitors-Low-Voltage-Dielectric-Capacitor/dp/B07X5XTDPB) |
| TUOFENG 22 AWG wire (straps + D/C + A-input) | 3–5 | [link](https://www.amazon.com/Electric-Flexible-Silicone-different-Electronic/dp/B07G2JWYDW) |
| BTF-LIGHTING 4-pin JST-SM, 5-pack | 4 | [link](https://www.amazon.com/BTF-LIGHTING-Connector-Extension-Solderless-Controller/dp/B0DCB76FP4) |
| 220 Ω resistors ×6 | 4 | already owned |
| Cermant 1×40 extra-tall female stacking header, 10-pack | 5 | [link](https://www.amazon.com/Cermant-Connector-Female-Stacking-Header%EF%BC%8CNeedle/dp/B0CJNF94HW) |
| Gikfun 1×40 breakaway male header pins, 10-pack | 5 | [link](https://www.amazon.com/Gikfun-2-54mm-Single-Breakaway-Arduino/dp/B00U8OCENY) |
| ESP32 UWB module | 5 | Makerfabs (sourced separately — not on Amazon) |
| DotStar/SK9822 strips | 5 | [Adafruit](https://www.adafruit.com/product/2242) / [BTF-LIGHTING SK9822 (Amazon)](https://www.amazon.com/BTF-LIGHTING-Individually-Addressable-Non-Waterproof-Separately/dp/B07BPX2KFD) |
| 2-pin JST-SM connector, M+F (10-pair) | 6 | [link](https://www.amazon.com/Pairs-Connector-Female-Cable-Pigtail/dp/B0F2MDYPXM) |

**Notes on a few picks:**
- The **inline fuse holder 5-pack** ships with an assortment that covers your 5 A, 2 A, and the possible 1 A downgrade, so it doubles as your fuse supply.
- The **1000 µF cap** is rated 25 V, comfortably above your 16 V+ requirement.
- The **SN74HCT245N** Amazon listing is sold in multiple pack sizes from the same authentic seller — 1-piece, 5-pack, and 10-pack were all confirmed available. `final_shopping_list.md` uses the 5-pack for spares; grab whichever quantity suits you.
- The **22 AWG TUOFENG** wire found on Amazon is flexible silicone stranded. Your solder order calls for *solid* straps in Phase 3/5; stranded works fine if tinned, but if you want stiff self-supporting straps, buy a small spool of 22 AWG *solid* separately.
- The **ESP32 UWB module** (Makerfabs) isn't carried on Amazon — order it direct as you'd planned.

The only thing worth adding to your cart that isn't already on the list is the **20-pin DIP socket** for the level shifter.
