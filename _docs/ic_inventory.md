# IC Inventory

Every IC (and RF module, treated the same way here) used or planned in the design, one row
per distinct part. **Placed** rows are pulled directly from the live schematic's own
Value/MPN/Manf/Footprint/LCSC properties — not re-derived — so if this ever disagrees with
the `.kicad_sch` files, the schematic is the source of truth, not this doc. **Planned** rows
are a specific real part chosen by LCSC price/stock/brand reputation, not yet placed.
Pricing/stock snapshotted 2026-09-20 — LCSC prices and stock move constantly, re-check before
actually ordering.

Brand-preference rule applied throughout (per your stated criteria): reputable global brands
(TI, onsemi, NXP, Nexperia, Microchip, u-blox, etc.) preferred over Asian-market-only brands
when the price point is reasonable (under ~$1–2) and the reputable option isn't meaningfully
worse on stock. Cheaper Asian-market brands are kept where the price gap is real and material
(e.g. WCH's USB-UART/hub parts, already placed — your own call, and the right one: several
dollars cheaper than CP2102N/FT-class equivalents for equivalent function).

## Placed (in the schematic today)

| Function | Qty | Manufacturer | MPN | Package | LCSC Part # | LCSC Price (unit) | LCSC Stock | Refs |
|---|---|---|---|---|---|---|---|---|
| Battery fuel gauge/protector | 1 | Maxim (Analog Devices) | MAX17320G22+ | TQFN-24 (4×4mm) | `C2914309` | $5.86 (1pc) | 376 | U5 |
| Battery/PowerPole input mux (VCOMP) | 3 | TI | TPS2121RUXR | VQFN-12 (2.5×2.0mm) | `C485916` | $1.06 (1pc) | 39,768 | U1, U2, U3 |
| Logic AND gate (latch wake-OR support) | 1 | TI | SN74LVC1G08DBVR | SOT-23-5 | `C7666` | $0.049 (10+; MOQ 10) | 88,180 | U4 |
| Buck converter, 5V/3.3V pre-regulation | 2 | TI | TPS563201DDCR | SOT-23-6 | `C116592` | $0.063 (10+; MOQ 10) | 73,660 | U6, U7 |
| LDO, 5V rail cleanup | 1 | HANSCHIP Semiconductor | LM1085IS-5.0RG | TO-263-3 | `C5145336` | $0.30 (1pc) | 980 | U8 |
| LDO, 3.3V rail cleanup + USB-side 3.3V | 2 | Advanced Monolithic Systems | AMS1117-3.3 | SOT-223-3 | `C6186` | $0.22 (5pc, MOQ 5) | 881,175 | U9, U12 |
| MCU module | 1 | Espressif | ESP32-S3-WROOM-1-N8R8 | RF module | `C2913201` | $5.04 (1pc) | 722 | U10 |
| USB hub | 1 | WCH | CH334R | QSOP-16 | `C4154405` | $0.57 (1pc) | 19,833 | U11 |
| USB-UART bridge | 1 | WCH | CH340C | SOIC-16 | `C84681` | $0.59 (1pc) | 106,811 | U13 |
| RTC | 1 | NXP | PCF8563T/5,518 | SOIC-8 | `C7440` | $0.63 (1pc) | 13,425 | U14 |

**Flag on the LDO (U8):** HANSCHIP Semiconductor is a smaller Chinese manufacturer, not in
the same reputation tier as the rest of this list — an existing pick from before this
inventory pass, not something I chose against your stated preference. A TI or onsemi
LM1085-class part would fit your stated brand rule better if you want to swap it; not
urgent, just flagging since this doc's whole point is surfacing exactly this kind of thing.

## Planned (chosen part, not yet placed)

| Function | Qty | Manufacturer | MPN | Package | LCSC Part # | LCSC Price (unit) | LCSC Stock | Notes |
|---|---|---|---|---|---|---|---|---|
| GPS/GNSS module | 1 | u-blox | MAX-M10S-00B | Module | `C4153167` | $9.81 (1pc) | 763 | Already the specifically-named part in `controller_platform.md`; no real "cheaper brand" alternative exists for this exact chipset/grade. |
| VHF RF transceiver module | 1 | G-NiceRF | SA818S-V | Module | `C51897911` | $8.98 (list) | **0 — out of stock, "Notify Me" only** | Already the specifically-named module. G-NiceRF is the original/canonical source for this exact product, not a clone situation — but it's currently unavailable at LCSC; worth checking their other listings (e.g. the `-U` UHF variant is a different band, not a substitute) or an alternate distributor before this becomes blocking. |
| I2S audio DAC | 1 | TI | PCM5102APWR | TSSOP-20 | `C107671` | $0.83 (1pc) | in stock | Chosen over NXP UDA1334ATS/N2,118 (`C494494`, ~$0.70, also a fine reputable-brand option) — PCM5102A is the more commonly used/documented part for this exact "simple I2S playback" use case, ~$0.13 more for meaningfully more community/example-code support. Revisit if cost trimming matters more than that. |
| I2C GPIO expander (status LEDs) | 1 | TI | PCA9555PWR | TSSOP-24 | `C2864778` | $0.73 (1pc) | 14,239 | Chosen over NXP PCF8574T/3,518 (`C7605`, ~$1.17, 2,991 stock) — cheaper, better stocked, and 16-bit vs. 8-bit for more future headroom, from an equally reputable brand. Clear win on every axis. |
| I2C EEPROM (config storage) | 1 | Microchip | AT24C32D-SSHM-T | SOIC-8 | `C60583` | $0.154 (1pc) | in stock | Already the specifically-named part family in `controller_platform.md`; Microchip (ex-Atmel) is the original/canonical brand for this exact device, trivially cheap regardless of brand. |
| Dual retriggerable monostable (TX-safety timeout) | 1 | Nexperia | 74HC123D,653 | SOIC-16 | `C5597` | $0.33 (5pc, MOQ 5) | 24,075 | Genuine dual-section part (needed — one section for the 2-min PTT timeout, the other reserved as the hold-to-power-off hardware failsafe, see `open_items.md`). TI's own dual variant (CD74HCT123E) was out of stock at LCSC when checked; TI's in-stock alternative (SN74LVC1G123) is single-section only and doesn't fit. Nexperia is NXP's spun-off discrete/logic division — same reputation tier. |
| Forward-power fault comparator | 1 | onsemi | LM393DR2G | SOIC-8 | `C7955` | $0.032 (1pc) | 209,190 | Clear pick — cheapest *and* best-stocked *and* a named-preferred brand, no tradeoff. |
| PTT-keying optocoupler | 1 | *unconfirmed* | *(PC817-class, single-channel)* | SOP-4/DIP-4 | — | ~$0.02–0.06 depending on brand | varies | **Not finalized** — couldn't find an in-stock single-channel part from a top-tier brand (Sharp's genuine single-channel SOP-4 variant, `C4075`, is currently out of stock; Toshiba's in-stock TLP621 listing at LCSC is a quad-channel DIP-16, wrong fit). All PC817-class parts are a few cents regardless of brand, so the usual "$1–2 threshold" reasoning barely applies here — recommend just confirming stock on a reputable single-channel part at actual build time rather than locking one in now. |

## Address/part-number verification note

Placed-row data is copied from the schematic's own properties (already individually
datasheet/LCSC-verified earlier in this project's history — see
`_docs/sourcing/task1_lcsc_crossreference.md`), not re-verified here. Planned-row parts were
each checked against their real LCSC listing (price, stock, manufacturer) this pass, not
assumed — same standard as the rest of this project's sourcing decisions. Prices and stock
are a snapshot; re-check before ordering, especially the SA818S-V (currently 0 stock) and the
optocoupler (not yet finalized).
