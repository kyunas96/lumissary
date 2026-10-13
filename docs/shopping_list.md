# LED Brain Hemisphere Build — Final Shopping List

## Board & Core Components
| Item | Link |
|---|---|
| Double-sided, plated-through-hole protoboard, 12x18cm | [Amazon](https://www.amazon.com/Double-Sided-Universal-Packs-12x18/dp/B08WJBS4HK) |
| Juried Engineering SN74HCT245N, DIP-20 (5-pack) | [Amazon](https://www.amazon.com/Juried-Engineering-SN74HCT245N-Transceivers-Breadboard-Friendly/dp/B08FZN8WM1) |
| Ganasome 20-pin DIP IC socket, 2.54mm pitch, through-hole (20-pack) — protects the chip from iron heat, lets you swap it without desoldering | [Amazon](https://www.amazon.com/Ganasome-Spacing-Machine-Through-Hole-20-Pack/dp/B0CX8XM2YW) |
| Cermant Female Stacking Headers, extra tall | Amazon |
| 20pcs Male Breakaway Header Pins | Amazon |

## Power
| Item | Link |
|---|---|
| Fytoo 2S LiPo Battery, 2000mAh, 35C, XT30 plug, built-in USB charging | [Amazon](https://www.amazon.com/Battery-2000mAh-Charging-Quadcopter-Aircraft/dp/B0FJFMC34Q) |
| XT30 Connector Pigtail with Pre-Fitted 15A Fuse, M+F, 14AWG (battery-end fuse) | Search "XT30 fused pigtail 15A" on Amazon |
| FEICHAO 8A UBEC, 7V-25.5V in, 2-6S (main LED bus) | [Amazon](https://www.amazon.com/FEICHAO-Output-7V-25-5V-Switch-Helicopters/dp/B07DD9L6P6) |
| Yuenhoang 3A UBEC, 2S-12S, 5V/6V switchable (ESP32 rail) | [Amazon](https://www.amazon.com/Yuenhoang-Adjustable-Shielding-Antijamming-Switchable/dp/B0D2NHBKJJ) — ⚠️ see note below |
| Inline fuse holder 5-pack, 12AWG, w/ assorted ATC/ATO fuses incl. 2A/5A (VicVan — heavier gauge and wider fuse range than the original pick; need 2 on-board holders — 5A for the UBEC branch, 2A for the Small BEC branch) | [Amazon](https://www.amazon.com/VicVan-Pack-Inline-Fuse-Holder/dp/B0CLK77GYN) |

## Connectors
| Item | Link |
|---|---|
| Frienda 20-Pair XT30 Connectors, bare solder-cup, M+F, w/ heat shrink (covers the splitter-to-regulator adapters, the two on-board XT30 ports, and general use — 20 pairs is well beyond what's needed) | [Amazon](https://www.amazon.com/Frienda-Connectors-Female-Pieces-Battery/dp/B0GQLMPWRD) |
| XT30 "1 Female to 2 Male" Y-splitter | [Amazon](https://www.amazon.com/Parallel-Connector-1-Female-Multicopter-Quadcopter/dp/B08GP3LSY3) |
| BTF-LIGHTING 4-Pin JST SM, 20AWG, 1m — SK9822-compatible (5-pack) | [Amazon](https://www.amazon.com/BTF-LIGHTING-Connector-Extension-Solderless-Controller/dp/B0DCB76FP4) |
| 2-pin JST connector, M+F (for the board's fan port — always populated on the board, whether or not a fan is actually connected) | Search "2 pin JST connector kit" on Amazon |

## Wire
| Item | Link |
|---|---|
| TUOFENG 18 AWG Stranded Wire | [Amazon](https://www.amazon.com/TUOFENG-Wire-Silicone-Insulated-Stranded/dp/B07G744V5Z) |
| TUOFENG 22 AWG Solid Core Hookup Wire | [Amazon](https://www.amazon.com/TUOFENG-Wire-Solid-Different-Colored/dp/B07TX6BX47) |

## Capacitors
| Item | Link |
|---|---|
| Electrolytic capacitor assortment, 0.1µF-1000µF, 10-50V (use 1000µF @ 16V+ as bulk cap) | [Amazon](https://www.amazon.com/ALLECIN-Electrolytic-Capacitor-Assortment-Kit/dp/B0C1VBXCQM) |
| Ceramic capacitor assortment, 0.1-10µF (use 0.1µF as decoupling cap) | [Amazon](https://www.amazon.com/BOJACK-Capacitor-Multilayer-Monolithic-Assortment/dp/B09YV2659V) |

## Already Have / Sourcing Separately
- 220Ω resistors
- ESP32 UWB module (Makerfabs)
- Adafruit DotStar LED strips (product 2328, SK9822) — buying directly from Adafruit

## Build Notes
- Solder XT30 joints with the mating connector plugged in to keep pins straight; pre-tin wire and pre-fill the solder cup separately before joining to limit heat soak into the plastic housing.
- Slide heat shrink onto the wire *before* soldering.
- Lead-free rosin core solder is fine throughout; run the iron ~350-380°C for clean joints, especially on the XT30 solder cups.

## Availability Check
- Confirmed direct, in-stock links for most items above.
- **Yuenhoang UBEC**: this exact listing has several near-identical sibling variants (different voltage-jumper options — 5V/6V vs 5V/12V, etc.) sold under very similar titles. Double-check at checkout that you're getting the **5V/6V** version and that it's not showing "Currently Unavailable" — if it is, a same-family variant (5V/12V, still switchable to 5V) is a safe substitute.
- **XT30 fused pigtail and 2-pin JST connector**: added to this list as generic search terms rather than confirmed direct links — verify a specific listing's stock status before ordering, same as with any other item here.
- The fan itself (e.g. a 30mm 5V DC fan) is intentionally **not** on this list — the board's fan port is built to be populated regardless, but connecting an actual fan stays optional and can be added later without needing anything else from this list.
- Everything else showed no out-of-stock signals as of the last check.
