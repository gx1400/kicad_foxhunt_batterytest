# Fox Hunt Controller — Hardware Design

Custom amateur radio fox hunt (hidden transmitter) controller. Built around an ESP32-C3, u-blox GPS module, and SA818S 2m RF module, with a WiFi-based web UI for configuration, RTOS-based scheduling, RTC, GPS power-switchable via IO, TX watchdog/lockout, and APRS output capability. Firmware developed in PlatformIO. Boards fabricated and assembled via JLCPCB (including their SMT assembly service). Planned as an iterative multi-revision build, proving out core software before locking down hardware.

**Note:** this initial revision is a dev/learning board — built to test out the design, get hands-on with the parts and tooling, and work out mistakes before committing to a more polished revision.

This document covers the power/battery management hardware design (`foxhunt1.kicad_sch`) as worked out so far.

## Power Architecture Overview

Two independent power sources feed the board, combined safely before regulation:

1. **2S1P 18650 Li-ion pack** (4.2V/cell max, 8.4V max stack), protected and fuel-gauged by a MAX17320.
2. **External 12–14.5V input** (lead-acid or LiFePO4), arriving via Anderson Powerpoles, fused.

These two sources are **fully isolated from each other on the charge path** — the 18650 pack is charged externally/separately, not through this circuit. This board's MAX17320 section handles protection and fuel gauging only. The two sources' *outputs* are combined downstream via ideal-diode ORing rather than sharing a charge path.

```
18650 pack ──[MAX17320 protector]── +7.5V ──┐
                                              ├─[ORing FETs]── VRAW ──┬─[Buck+LDO]── 5V  (SA818S)
12–14.5V input ──[fuse]── +12V ──────────────┘                       └─[Buck+LDO]── 3.3V (ESP32-C3, GPS)
```

## 1. Battery Protection & Fuel Gauge — MAX17320

- 2S1P 18650 configuration. CELL2/CELL3 pins shorted to CELL1 (not BATTS) — matches the "2×RBAL" balancing path used for the top cell.
- **ModelGauge m5 EZ** fuel gauge mode — no per-cell characterization required.
- **Sense resistor:** 5mΩ, 4-terminal Kelvin (2512 package). Must be explicitly configured in nonvolatile memory during the config wizard — the IC defaults to 2.5mΩ (RSenseSel code 1), which would silently halve every current/capacity reading if left unset.
- **Cell balancing:** RBAL1 (CELL1) = 150Ω, RBAL4 (BATTS) = 150Ω. Bottom cell ≈ 26mA, top cell ≈ 13.6mA (routes through both RBAL1 and RBAL4 in series).
- **Zero-volt charge recovery:** RZVC = 820Ω, sized for ~10mA recovery current at an assumed 8.4V charger CV rail.
- **Permanent-failure fuse:** 3-terminal fuse (Littelfuse ITV4030L1212 or equivalent), heater triggered by PFAIL through an AO3400A N-FET.
- **CHG/DIS FETs:** back-to-back dual N-FET pair, gates on CHG/DIS, sources split — one referencing raw battery (IN side), one referencing the post-FET system node (PCKP side).
- **RIN:** 10Ω per datasheet spec, in series with IN.
- **PFAIL, unused comms/thermistor pins:** ALRT, SCL/OD, SDA/DQ, TH2–TH4 left as no-connects; only TH1 populated.

### Ground domain split (GND vs. GNDREF)

Two separate ground nets, bridged **only** by the current-sense resistor (R3):

- **GNDREF** (battery-referenced island): U1's GND pin, CSP, battery negative terminal, IN/CP/REG2/REG3-area bypass caps.
- **GND** (system-referenced): CSN, all downstream regulator grounds, output-side bypass caps.

This split is what makes R3 actually measure current rather than being bypassed by a parallel ground path. Confirmed correct in the current netlist.

## 2. Dual-Input Power Combining

- **LM74610-Q1** ideal-diode controllers, one per input branch, each driving an external N-FET (AO3400A) as a near-lossless "smart diode."
- Chosen over passive Schottky diode-ORing specifically to minimize voltage drop and power dissipation (roughly 9x lower loss at ~2A vs. a Schottky pair).
- VCAP on each LM74610: 1µF, 16V, X7R.
- Extensive solder-jumper isolation points (JP2–JP10) throughout this section and the regulation stages, letting each stage — battery branch, 12V branch, buck input, buck-to-LDO handoff — be bench-tested independently before trusting the full chain.

## 3. Regulation — VRAW → 5V and 3.3V Rails

Two **independent** buck (TPS563201) + linear-LDO cascades, rather than one shared intermediate rail, so each LDO stage runs close to its own output voltage and wastes minimal power as heat.

| | 5V rail (→ SA818S) | 3.3V rail (→ ESP32-C3, GPS) |
|---|---|---|
| Buck stage | TPS563201, 6V intermediate | TPS563201, 4.3V intermediate |
| Inductor | 3.3µH (Chilisin MHCI06030-3R3M-R8) | 3.3µH (same part) |
| Feedback (R_top / R_bottom) | 68.1kΩ / 10kΩ | 46.4kΩ / 10kΩ |
| Output caps | 2× 22µF/16V X7R ceramic | 2× 22µF/16V X7R ceramic |
| LDO | LM1085-5.0 | AMS1117-3.3 |
| LDO output cap | 22µF/16V **aluminum electrolytic** (~200mΩ ESR) | 0.1µF ceramic + 22µF/16V aluminum electrolytic (~200mΩ ESR) |
| Design current | 2A (SA818S TX peak ~750mA + margin) | 1A (real worst-case ~470mA) |
| Peak inductor current | ~2.9A | ~1.8A |

**Note on LDO output caps:** both LM1085 and AMS1117 rely on the output cap's ESR for loop stability (opposite requirement from the buck stage's low-ESR-ceramic-friendly D-CAP2 topology) — a low-ESR ceramic reused from the buck section risks oscillation. Aluminum electrolytics in the ~100–300mΩ ESR range satisfy both LDOs' requirements.

**Planned addition:** optional 200mA test-load jumpers with an in-line DMM measurement header on each LDO output, for bench current verification ahead of populating the real downstream loads.
- 5V rail: 24.9Ω, 2W
- 3.3V rail: 16.5Ω, 1W

## Downstream Loads (planned, not yet on this board)

| Device | Rail | Typical | Peak |
|---|---|---|---|
| SA818S (2m RF module) | 5V | ~60mA RX | ~750mA TX |
| ESP32-C3 | 3.3V | ~20–80mA | ~300–400mA (radio TX burst) |
| u-blox GPS | 3.3V | ~25–45mA | ~50–70mA (acquisition) |

## Open Items / Next Steps

- Confirm GNDPWR solder-jumper flags are wired to the intended nets (not accidentally re-merging GND/GNDREF) — likely a KiCad global GND power-symbol mixup if it recurs.
- Populate downstream loads (SA818S, ESP32-C3, GPS) and re-verify rail current budgets against real hardware.
- Decide on always-on backup supply for GPS V_BCKP if warm-start fix times matter.
- First board bring-up: isolate each stage via JP2–JP10, verify independently, then re-bridge for full-system test.
