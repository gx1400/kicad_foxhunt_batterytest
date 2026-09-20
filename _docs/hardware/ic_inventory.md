# IC Inventory

Every IC (and RF module, treated the same way here) used or planned in the design.
Pricing/stock snapshotted 2026-09-20 — LCSC prices and stock move constantly, re-check
before actually ordering. Brand-preference rule applied throughout (per your stated
criteria): reputable global brands (TI, onsemi, NXP, Nexperia, Microchip, u-blox, etc.)
preferred over Asian-market-only brands when the price point is reasonable (under ~$1–2)
and the reputable option isn't meaningfully worse on stock. Cheaper Asian-market brands
are kept where the price gap is real and material (WCH's USB-UART/hub parts, already
placed — several dollars cheaper than CP2102N/FTDI-class equivalents for equivalent
function, a good call).

# Inventory

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
inventory pass, not something chosen against the stated brand preference. See the LDO —
5V Rail section below for a same-price-tier, reputable-brand swap option.

## Planned (chosen part, not yet placed)

| Function | Qty | Manufacturer | MPN | Package | LCSC Part # | LCSC Price (unit) | LCSC Stock | Notes |
|---|---|---|---|---|---|---|---|---|
| GPS/GNSS module | 1 | u-blox | MAX-M10S-00B | Module | `C4153167` | $9.81 (1pc) | 763 | Already the specifically-named part in `controller_platform.md`. |
| VHF RF transceiver module | 1 (DNP) | G-NiceRF | SA818S-V | Module | — | — | — | **DNP — sourced outside LCSC**, not part of the JLCPCB assembly BOM. See below. |
| I2S audio DAC | 1 | TI | PCM5102APWR | TSSOP-20 | `C107671` | $0.83 (1pc) | in stock | Chosen over NXP UDA1334ATS/N2,118 (~$0.70, also fine) for community/example-code support. |
| I2C GPIO expander (status LEDs) | 1 | TI | PCA9555PWR | TSSOP-24 | `C2864778` | $0.73 (1pc) | 14,239 | Cheaper, better-stocked, and 16-bit vs. NXP PCF8574's 8-bit — clear win. |
| I2C EEPROM (config storage) | 1 | Microchip | AT24C32D-SSHM-T | SOIC-8 | `C60583` | $0.154 (1pc) | in stock | Already the specifically-named part family; Microchip (ex-Atmel) is the canonical brand. |
| Dual retriggerable monostable (TX-safety timeout) | 1 | Nexperia | 74HC123D,653 | SOIC-16 | `C5597` | $0.33 (5pc, MOQ 5) | 24,075 | Genuine dual-section part — needed, since one section is reserved for the hold-to-power-off hardware failsafe. |
| Forward-power fault comparator | 1 | onsemi | LM393DR2G | SOIC-8 | `C7955` | $0.032 (1pc) | 209,190 | Cheapest, best-stocked, and a named-preferred brand — no tradeoff. |
| PTT-keying optocoupler | 1 | *unconfirmed* | *(PC817-class, single-channel)* | SOP-4/DIP-4 | — | ~$0.02–0.06 | varies | Not finalized — see its own section below. |

# Alternatives

**Compatibility rating** used throughout this section: three sub-scores, 0–2 each, summed
to a /6 total.

- **Pin/Footprint** — 2 = identical footprint *and* pinout (true drop-in); 1 = same
  package family but a pin function or pitch differs; 0 = different package entirely.
- **Electrical/Functional** — 2 = same core function and comparable specs, no circuit
  changes needed; 1 = same function, some parameter difference needing minor passive/BOM
  rework (different divider resistors, different caps, etc.); 0 = different topology,
  needs a real redesign.
- **Sourcing** — 2 = well-stocked (>1,000 units) *and* a reputable brand; 1 = one of the
  two but not both; 0 = low stock and/or an unverified/no-name brand.

| Total | Tier |
|---|---|
| 5–6 | **Drop-in** — swap with no board changes |
| 3–4 | **Compatible, minor rework** — same footprint or function, expect small BOM/value changes |
| 1–2 | **Functional alternative, redesign needed** — different footprint or topology |
| 0 | **Not recommended** |

---

## Battery Fuel Gauge / Protector

2S–4S Li-ion pack protection (over/under-voltage, over-current, temperature) plus
ModelGauge m5 EZ fuel gauging, driving external high-side N-FETs for CHG/DIS control.
This is a fairly specialized combined function — genuine drop-in alternates are scarce.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **Maxim (ADI)** | **MAX17320G22+** | **TQFN-24** | **`C2914309`** | **$5.86** | **376** | — |
| Alt. 1 | Maxim (ADI) | MAX17205 | Different pin-count | *not priced this pass* | — | — | 2/6 — same family, but a monitor/balance IC without the integrated protector; needs a separate protection path. Different pinout, partial function match, sourcing unconfirmed. |
| Alt. 2 | TI | BQ76942 | Different package | *not priced this pass* | — | — | 1/6 — analog front-end architecture, not a fuel gauge; would need a separate gauge IC alongside it. Real redesign, not a substitution. |

**Bottom line:** no real drop-in exists for this exact part at this price/stock tier;
if `MAX17320` ever becomes unavailable, budget for a redesign, not a swap.

## Power Mux (VCOMP)

2:1 priority power multiplexer — automatically selects and seamlessly switches between
two input sources via ideal-diode operation. One instance combines the battery vs.
PowerPole 12V input into VRAW; two more instances mux USB vs. battery-regulated output on
the 5V and 3.3V rails.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **TI** | **TPS2121RUXR** | **VQFN-12** | **`C485916`** | **$1.06** | **39,768** | — |
| Alt. 1 | TI | TPS2116 | Different package | *not priced this pass* | — | — | 2/6 — newer-generation TI part, but only rated 1.6–5.5V input; fine for the 3.3V/5V rail-mux instances, **not usable** for the raw battery-mux instance (VRAW reaches ~14.4V). Different footprint too. |

**Bottom line:** worth checking `TPS2116` specifically for the two rail-mux instances
(cheaper, lower Rds(on), same brand) as a *partial* swap, but the battery-mux instance
needs `TPS2121`'s higher voltage headroom regardless.

## Logic AND Gate (latch wake-OR support)

Single 2-input AND gate supporting the VRAW soft-latch wake network.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **TI** | **SN74LVC1G08DBVR** | **SOT-23-5** | **`C7666`** | **$0.049 (10+)** | **88,180** | — |
| Alt. 1 | Nexperia | 74LVC1G08GW,125 | SOT-23-5 (TSOP-5) | `C12512` | $0.067 (1pc) | 13,365 | 6/6 — industry-standard single-gate pinout, identical package, same reputable-brand tier. True drop-in. |

## Buck Converter (VRAW → 5V/3.3V pre-regulation)

Synchronous step-down converter, D-CAP2 control, SOT-23-6. Two instances (5V and 3.3V
pre-regulation stages, both feeding their respective LDO).

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **TI** | **TPS563201DDCR** | **SOT-23-6** | **`C116592`** | **$0.063 (10+)** | **73,660** | — |
| Alt. 1 | Diodes Incorporated | AP62200WU-7 | SOT-23-6 (similar) | `C1323282` | $0.091 (est.) | 41,605 | 4/6 — similar 6-pin footprint and function, but 2A rated vs. TPS563201's 3A (this design's actual draw is 2A per `regulation.md`, so likely fine in practice) — different control architecture/switching frequency means it's not a blind pin-swap, verify loop stability with the existing LC values. |
| Alt. 2 | Richtek | RT8059 | **SOT-23-5** | — | — | — | 1/6 — wrong pin count (5 vs. 6), excluded as a footprint match despite similar function. |
| Alt. 3 | Monolithic Power Systems | MP2315 | **TSOT-23-8** | — | — | — | 1/6 — wrong package entirely. |

## LDO — 5V Rail

Linear regulator, TO-263, cleaning up the 5V buck's output for the SA818S.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **HANSCHIP Semiconductor** | **LM1085IS-5.0RG** | **TO-263-3** | **`C5145336`** | **$0.30** | **980** | — |
| Alt. 1 | TI | LM1085IT-5.0/NOPB | **TO-220**, not TO-263 | `C9721` | $2.32 | 0 (out of stock) | 2/6 — genuine TI part, but wrong package *and* out of stock at LCSC right now. Not a practical swap today. |
| Alt. 2 | TI | LM1084IRTG3-5.0 (5A version) | TO-263-3 | *not priced this pass* | — | — | 5/6 — TI states this shares LM317's pinout, same as LM1085's; more current headroom than needed (5A vs. this design's 2A). Likely the best reputable-brand swap if you want to move off HANSCHIP, pending a price/stock re-check. |
| Alt. 3 | Advanced Monolithic Systems | AMS1085CM-5.0 (if stocked; only the -3.3 variant was confirmed this pass) | TO-263 | *not priced this pass* | — | — | 4/6 — same brand family already used elsewhere in this design (U9/U12), genuinely pin-compatible per AMS's own datasheet claims; voltage variant not independently confirmed in stock this pass. |

## LDO — 3.3V Rail (main + USB-side)

Linear regulator, SOT-223, fixed 3.3V — used for both the main 3.3V rail cleanup and the
USB-VBUS-derived logic-only 3.3V path.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **Advanced Monolithic Systems** | **AMS1117-3.3** | **SOT-223-3** | **`C6186`** | **$0.22 (5pc)** | **881,175** | — |
| Alt. 1 | TI | LM1117IMPX-3.3/NOPB | SOT-223-3 | `C23984` | $0.134 (1pc) | in stock | 6/6 — genuinely pin-to-pin and electrically compatible per TI's own datasheet (same SOT-223 1117-family pinout); cheaper *and* a more reputable brand. Best swap candidate in this whole doc. |
| Alt. 2 | onsemi | NCP1117ST33T3G | SOT-223-3 | `C26537` | $0.099 (1pc) | in stock | 6/6 — same 1117-family drop-in pinout, cheaper still, onsemi is equally reputable. |
| Alt. 3 | Richtek | RT9193 | **SOT-23-5** | — | — | — | 1/6 — different, smaller package; not a footprint match despite similar low-dropout function. |

**Bottom line:** this is the clearest "should probably swap" in the whole inventory — two
reputable-brand, cheaper, genuinely pin-compatible options exist.

## MCU Module

ESP32-S3 with 8MB flash + 8MB octal PSRAM (PSRAM matters for concurrent WiFi + audio +
SD workload, see `controller_platform.md`).

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **Espressif** | **ESP32-S3-WROOM-1-N8R8** | **Module, 56 pads** | **`C2913201`** | **$5.04** | **722** | — |
| Alt. 1 | Espressif | ESP32-S3-MINI-1-N8 | Smaller module, different footprint/pad count | `C2913206` | $3.14 | 2,084 | 2/6 — cheaper and better-stocked, but a genuinely different footprint (smaller module, fewer castellated pads) — real board rework, not a swap. Also lacks octal PSRAM in the plain `-N8` variant (check `-N8R2`/similar if PSRAM is still wanted). |
| Alt. 2 | Espressif | ESP32-S3-WROOM-2-N32R8V | Same WROOM-2 footprint family, larger flash/PSRAM | `C3021270` | $13.74 | in stock | 3/6 — likely closer to pin-compatible (same WROOM form factor lineage) but meaningfully more expensive for capability this design doesn't need. Not independently confirmed pin-identical to WROOM-1 this pass. |

**Bottom line:** this module's footprint is effectively a one-off within this design —
no meaningfully cheaper drop-in exists; the MINI variant is the real cost/size lever, but
it's a layout change, not a swap.

## USB Hub

Splits one USB-C connection between the MCU's native USB and the USB-UART bridge.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **WCH** | **CH334R** | **QSOP-16** | **`C4154405`** | **$0.57** | **19,833** | — |
| Alt. 1 | Microchip | USB2513B-AEZC-TR | Different package/pinout | `C219466` | $1.25 | reported out of stock | 1/6 — reputable brand, but ~2x the price *and* not currently stocked; also a 3-port hub architecture, not a direct pin match. |
| Alt. 2 | Microchip | USB2517-JZX | Different package/pinout, 7-port | *(see below)* | $2.52 | in stock | 1/6 — genuine global brand and in stock, but ~4.4x the price for a 7-port hub this design doesn't need — the WCH price/function fit is clearly better here. |

**Bottom line:** this is the case referenced in your own message — WCH is the right call;
the "reputable brand" alternates cost multiples more for capability this design doesn't
use.

## USB-UART Bridge

Standalone USB-to-UART bridge for a stable serial console across MCU resets (native-USB
CDC re-enumerates on every reset, this doesn't).

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **WCH** | **CH340C** | **SOIC-16** | **`C84681`** | **$0.59** | **106,811** | — |
| Alt. 1 | Silicon Labs | CP2102N-A02-GQFN28R | **QFN-28**, different footprint | `C964632` | $1.02–1.22 | 40,646 | 2/6 — the "professional-grade" alternative (better driver polish, configurable GPIOs), but ~2x the price and a completely different QFN footprint — real rework, not a swap. |

**Bottom line:** same story as the hub — WCH wins on price for equivalent function;
CP2102N would be the pick only if driver/GPIO features become worth the cost and layout
change.

## RTC

I2C real-time clock/calendar, external 32.768kHz crystal, coin-cell-backed.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **NXP** | **PCF8563T/5,518** | **SOIC-8** | **`C7440`** | **$0.63** | **13,425** | — |
| Alt. 1 | NXP | PCF8523T/1,118 | SOIC-8 | `C2651516` | $0.56 | in stock | 3/6 — same brand, same 8-pin SO package, newer/lower-power design — but a *different* pin function layout and a 1MHz vs. 400kHz I2C spec, so not pin-identical despite matching package. Would need re-checking the crystal load-cap requirement too. |
| Alt. 2 | Micro Crystal | RV-3028-C7-32.768kHz-1ppm-TA-QC | Different package | `C3019759` | $1.40 | in stock | 2/6 — far better accuracy (±1ppm vs. PCF8563's crystal-dependent accuracy) and only 45nA backup draw, but a different footprint and meaningfully pricier — this design already accepts PCF8563's accuracy since GPS periodically disciplines it, so the upgrade isn't needed, just noting it exists. |
| Alt. 3 | Maxim (ADI) | DS3231 | Different package, integrated TCXO | — | ~$9–10.50 | — | 1/6 — already considered and rejected in `controller_platform.md` for cost; included here only for completeness. |

## GPS/GNSS Module

Standard-precision GNSS, UART+I2C+PPS, external active antenna via SMA.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **u-blox** | **MAX-M10S-00B** | **Module (LCC)** | **`C4153167`** | **$9.81** | **763** | — |
| Alt. 1 | u-blox | NEO-M9N-00B | Larger module, different footprint | `C5119087` | $15.96 | **2 — effectively out of stock** | 2/6 — same brand, but a different (larger, older-generation) module footprint, more expensive, and barely stocked. Not a good swap right now. |
| Alt. 2 | Quectel | L76-K | Different footprint | — | ~$12.50–15.90 | — | 2/6 — reputable global brand (Quectel), meter-level accuracy similar class, but different module footprint and pricier — real board rework, not a swap. |

**Bottom line:** `MAX-M10S` is already the best price/stock/capability fit in its class;
the alternates are strictly worse on at least two axes each.

## VHF RF Transceiver Module

The onboard SA818S footprint (and the shared audio/PTT path also serving an external HT
jack) — see `controller_platform.md`.

**Status: DNP.** Confirmed not sourced through LCSC — this footprint won't be part of the
JLCPCB assembly BOM. When this footprint is added to the schematic, place it with
`(dnp yes)` from the start rather than defaulting to populated and fixing it later. No
LCSC part number, price, or stock tracked here since it's out of scope for that supply
chain; source directly from G-NiceRF or another channel outside this project's normal
LCSC/JLCPCB flow.

| | Manufacturer | MPN | Package | Notes |
|---|---|---|---|---|
| **Selected (DNP)** | **G-NiceRF** | **SA818S-V** | **Module** | Not sourced via LCSC — see status note above. |

No alternates researched — moot while this stays DNP and outside the LCSC supply chain.

## I2S Audio DAC

Simple stereo I2S DAC for voice-ID/CW-tone audio, feeding the SA818S/HT MIC path.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **TI** | **PCM5102APWR** | **TSSOP-20** | **`C107671`** | **$0.83** | in stock | — |
| Alt. 1 | NXP | UDA1334ATS/N2,118 | Different package, similar pin-count | `C494494` | $0.70 | in stock | 3/6 — equally reputable brand, cheaper, same core I2S-DAC function — but a different physical package/pinout (not independently confirmed pin-identical), so likely a footprint change even though electrically a straightforward swap. |
| Alt. 2 | TI | PCM5122 | Different, larger pinout | — | — | — | 2/6 — same family, more features (headphone amp, etc.) this design doesn't need, different pinout — not a drop-in. |

## I2C GPIO Expander

Drives the TX-active and heartbeat status LEDs from register writes instead of native
GPIOs (see `esp32s3_pinout.md`).

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **TI** | **PCA9555PWR** | **TSSOP-24** | **`C2864778`** | **$0.73** | **14,239** | — |
| Alt. 1 | NXP | PCF8574T/3,518 | **SOIC-8**, 8-bit | `C7605` | $1.17 | 2,991 | 2/6 — half the I/O (8-bit vs. 16-bit), smaller/different package, pricier, and capped at 100kHz I2C — strictly worse on every axis for this use case. |
| Alt. 2 | TI | PCA9535RGER | Different package (VQFN) | `C2873080` | *not priced this pass* | — | 4/6 — same 16-bit register model and TI brand, functionally near-identical to PCA9555 (minor pull-up-config register differences), but a different QFN footprint. |

## I2C EEPROM

32Kbit config storage — schedule, frequency, TX power, tone settings — independent of
the SD card.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **Microchip** | **AT24C32D-SSHM-T** | **SOIC-8** | **`C60583`** | **$0.154** | in stock | — |
| Alt. 1 | onsemi | CAT24C32YI-GT3 | SOIC-8 | `C94264` | $0.187 | in stock | 6/6 — the 24Cxx family pinout is an industry standard across every manufacturer; this is about as true a drop-in as exists in this whole document. |
| Alt. 2 | STMicroelectronics | M24C32-FMN6TP | SOIC-8 | `C2061453` | $0.079 | in stock | 6/6 — same standard pinout, cheapest of the three, equally reputable brand. |

**Bottom line:** all three are genuinely interchangeable; ST's is the cheapest if cost is
the deciding factor, otherwise the Microchip pick (already the specifically-named part in
the docs) is fine as-is.

## Dual Retriggerable Monostable (TX-safety timeout)

RC-timed hardware watchdog gating the PTT line — one section for the 2-minute TX
ceiling, the other reserved for the hold-to-power-off hardware failsafe.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **Nexperia** | **74HC123D,653** | **SOIC-16** | **`C5597`** | **$0.33 (5pc)** | **24,075** | — |
| Alt. 1 | TI | CD74HCT123E | SOIC-16 | `C2872602` | $0.39 (list) | **out of stock at LCSC when checked** | 4/6 — same standard 74x123 pinout (industry-standard across HC/HCT/LS variants) and a reputable brand, but unavailable right now — otherwise a clean swap. |
| Alt. 2 | onsemi | 74VHC123AMX | SOIC-16 | `C6636` | *not priced this pass* | 74 (very low) | 3/6 — same pinout family, reputable brand, but stock is too thin to rely on. |
| Alt. 3 | TI | SN74LVC1G123DCUR | **Single-section, SC-70/SOT** | `C123302` | $1.04 | 664 | 0/6 — wrong part shape entirely: single monostable, not dual. This design specifically needs the second section for the power-off failsafe — excluded despite being a genuine TI part. |

## Forward-Power Fault Comparator

Fast, ADC-independent "antenna fault / near-zero power" digital trip, paired with the
ADC-based power-level readback.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | **onsemi** | **LM393DR2G** | **SOIC-8** | **`C7955`** | **$0.032** | **209,190** | — |
| Alt. 1 | STMicroelectronics | LM393ADT | SOIC-8 | `C136033` | $0.137 | 42,510 | 6/6 — LM393 is an industry-standard dual-comparator pinout duplicated across essentially every analog manufacturer; genuine drop-in, just pricier than the onsemi pick. |
| Alt. 2 | TI | LM393DR | SOIC-8 | *not priced this pass* | — | — | 6/6 (expected) — same standard part, TI's own version; not independently priced this pass but functionally identical to the above. |

**Bottom line:** already the best pick — cheapest, best-stocked, reputable brand, and
the alternates only exist to confirm there's no compatibility risk if onsemi's ever runs
short.

## PTT-Keying Optocoupler

Isolates the MCU's PTT-intent signal from the SA818S/HT keying line.

| | Manufacturer | MPN | Package | LCSC Part # | Price | Stock | Compat. |
|---|---|---|---|---|---|---|---|
| **Selected** | *unconfirmed* | *(PC817-class, single-channel)* | SOP-4/DIP-4 | — | — | — | — |
| Alt. 1 | Sharp | PC817X3NSZW(C) | DIP-4, single-channel | `C4075` | $0.146 (5pc) | **out of stock** | 5/6 if in stock — genuine reputable-brand single-channel part, standard PC817 footprint; just unavailable right now. |
| Alt. 2 | Toshiba (via licensed manufacture) | TLP621-1 | DIP-4, single-channel | `C84568` (ISOCOM-branded listing) | $0.115 | in stock | 4/6 — standard PC817-equivalent footprint, single-channel, in stock — but this specific LCSC listing is ISOCOM's own manufacture of the TLP621 part number, not genuine Toshiba; still a reputable specialist opto brand, not a no-name clone. |
| Alt. 3 | UMW / generic | PC817C | DIP-4, single-channel | `C3008368` | $0.015 | 172,080 | 3/6 — correct footprint and function, huge stock, trivial cost — but an unverified/no-name-tier brand. Given the entire PC817 class costs a few cents regardless of brand, the "under $1–2" brand-preference threshold barely bites here; Alt. 2 is the more defensible pick if brand matters, Alt. 3 if it's purely a cost/stock decision. |

**Bottom line:** none of these is a slam dunk — recommend confirming stock on `TLP621-1`
(Alt. 2) at actual build time rather than locking in a choice now.

---

## Verification note

Placed-row data is copied from the schematic's own properties (already individually
datasheet/LCSC-verified earlier in this project's history — see
`_docs/sourcing/task1_lcsc_crossreference.md`), not re-verified here. Planned-row parts and
every alternate above were checked against LCSC search results this pass; a few alternates
are explicitly marked "not priced this pass" or "not independently confirmed" rather than
given invented numbers — treat those as leads to verify, not settled facts, before acting
on them.
