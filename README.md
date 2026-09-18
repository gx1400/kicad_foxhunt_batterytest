# Fox Hunt Controller — Hardware Design

Custom amateur radio fox hunt (hidden transmitter) controller, aimed at being a flexible, dynamic platform for a wide range of fox-hunt formats — not just a fixed single-mode beacon. Built around an ESP32-S3, u-blox GPS module (MAX-M10S), and SA818S 2m RF module, with a WiFi/BLE web UI for configuration and "found" logging, RTOS-based scheduling, RTC-driven wake and precise TX timing, GPS-disciplined timekeeping, and APRS output capability. Firmware developed in PlatformIO. Boards fabricated and assembled via JLCPCB (including their SMT assembly service). Planned as an iterative multi-revision build, proving out core software before locking down hardware.

**Note:** this initial revision is a dev/learning board — built to test out the design, get hands-on with the parts and tooling, and work out mistakes before committing to a more polished revision.

This document covers the power/battery management hardware (built, in `battery_18650_input.kicad_sch` / `battery_powerpole_input.kicad_sch` / `power_regulation.kicad_sch`, under the `foxhunt1.kicad_sch` root) and the controller/peripheral platform (planned, not yet in the schematic — see §4).

## Setup

This project depends on three external symbol/footprint libraries, vendored as git submodules under `libs/` so the project is self-contained and doesn't require any machine-specific KiCad library setup. Project-level `sym-lib-table` and `fp-lib-table` files reference them via `${KIPRJMOD}`, so they resolve automatically once the submodules are checked out — no manual library registration needed.

Clone with submodules:

```sh
git clone --recurse-submodules https://github.com/gx1400/kicad_foxhunt_batterytest.git
```

Or, if already cloned:

```sh
git submodule update --init --recursive
```

Everything else (standard KiCad libraries: `Device`, `power`, `Resistor_SMD`, etc.) ships with any stock KiCad 10 install — no extra setup required.

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

## 4. Controller & Peripheral Platform (Planned)

Design decisions for the controller section, worked out before schematic capture begins. Nothing in this section exists in the schematic yet — see [Open Items](#open-items--next-steps) for the path from here to a buildable revision. Target capabilities: audio/voice TX, CW ID keying, variable-duration/interval blipping, variable TX power, APRS TX with GPS (dithered position optional), randomized CTCSS tone, frequency hopping, a WiFi/BLE "found" log, RTC-scheduled precise-interval TX, and schedulable mode changes (e.g. more frequent TX the longer a fox goes unfound).

### 4.1 Power sequencing — discrete soft-latch

MAX17320 protects/fuel-gauges the battery but doesn't gate system power — that's a separate circuit. A discrete P-FET soft-latch sits between VRAW and the 3.3V/5V regulators' input:

- Button press (or RTC alarm) pulls the latch enable line, powering the board on.
- The MCU immediately self-latches by driving the enable line itself; the button then reads as a normal input.
- To power off, the MCU does its shutdown housekeeping and releases the latch — true zero-current off, not MCU sleep.
- **Wake sources are diode-ORed** into the enable line: button press *and* the PCF8563's open-drain alarm/interrupt output can each independently power the board up from full-off. This is what lets scheduled TX cycles happen without keeping the MCU awake (and draining the battery) between transmissions.

### 4.2 Timekeeping — PCF8563 RTC

**PCF8563** (I2C, external 32.768kHz crystal), not DS3231. DS3231's integrated TCXO gives better standalone accuracy (±2ppm) and needs no external crystal, but costs ~15-20x more (~$9-10.50 vs. ~$0.50 at JLCPCB) for a benefit this design doesn't need — accuracy is maintained instead by periodically disciplining the RTC from GPS whenever it has a fix, and drift over a single unattended event is forgiving either way.

Backup: **coin cell only** (CR20xx holder), no supercap option for the RTC specifically — a supercap can't hold time across the months/years this board may sit idle between events (self-discharges in days-to-weeks), where a coin cell holds ~10 years. Losing RTC time after long storage is an accepted, low-consequence tradeoff (resolved for free by GPS resync at next power-up).

PCF8563 is single-supply (no automatic VBAT switchover pin like DS3231) — backup requires an external diode-OR at VDD combining the main 3.3V rail and the coin cell, each through **low-leakage diodes** specifically (ordinary diode reverse leakage can be comparable to or exceed the RTC's own nanoamp-class backup draw).

### 4.3 GPS — u-blox MAX-M10S

UART+I2C, PPS output, external active antenna via **SMA connector** (not an integrated patch). Backup power kept **separate from the RTC's coin cell** — its own backup domain with both a coin-cell holder and a supercap footprint, user populates either/neither/both at assembly (same low-leakage diode-OR pattern as §4.2), feeding VBACKUP so ephemeris/RTC survive short sleeps for a fast warm-start instead of a full cold-start reacquisition.

### 4.4 MCU — ESP32-S3-WROOM-1-N8R8

Module (not bare chip) — this board already has one RF section to get right (SA818S); a second self-laid-out antenna-matching problem on the MCU's WiFi/BLE radio isn't worth it for this revision. N8R8 (8MB flash + 8MB PSRAM) — PSRAM matters more than extra flash for the concurrent WiFi + audio buffering + SD card workload. **Native USB** (no USB-UART bridge chip) for programming/debug — USB is data-only, no onboard charging (see §4.6). **BLE enabled** alongside WiFi — free with the same radio, gives a lower-power/faster-handshake alternative to the WiFi captive portal for the "found" log.

### 4.5 Audio / PTT path (shared by onboard SA818S and external HT)

The SA818S has **no digital baseband input** — its UART is control-only (frequency, squelch, volume, CTCSS/DCS). Voice, CW tone, and AFSK for APRS all have to arrive as analog audio on its MIC pin, so an audio DAC (I2S DAC → RC filter, or PWM+filter) is required regardless of "digital vs. analog" framing.

This is one shared subsystem serving two destinations, not two separate designs: the same DAC output and PTT-keying circuit (optocoupler) route to *both* the onboard SA818S footprint (populated later for eval, unlikely at initial fab per original scope) *and* a 3.5mm jack for an external HT, switchable/jumpered between the two.

### 4.6 Storage — SD card + EEPROM

- **SD card: SPI mode**, not SDIO. No audio *recording* planned, and audio *playback* (voice IDs, custom clips) is a light, bufferable workload (~32KB/s for 16kHz/16-bit mono) well within SPI-mode throughput — SDIO's speed and extra GPIO cost isn't needed.
- **Config storage: I2C EEPROM** (e.g. AT24C32D-class, 32Kbit), not FRAM — FRAM (FM24C/MB85RC families) isn't available in JLCPCB's catalog, and EEPROM's ~1M write-cycle life is a non-issue for settings that change occasionally (schedule, frequency, TX power, tone), not continuously. Lives on the same I2C bus as the RTC/fuel-gauge, independent of the SD card, so critical operating parameters survive a missing/corrupt card.

### 4.7 Onboard charging — none; USB powers logic only, via an isolated path

**No battery charging.** Investigated: TI **BQ25792** (I2C-configurable 1-4S buck-boost charger, in stock at JLCPCB) can charge the 2S1P pack directly from 5V USB without PD negotiation (internally boosts as needed) — a real, buildable option, not ruled out for cost/feasibility reasons. Decided against it for this revision anyway; charging stays external/separate as originally designed.

**USB VBUS is not left electrically dangling, though** — every USB connection carries 5V on VBUS regardless of intent, and if that were wired straight into the 3.3V rail or VRAW it would fight the battery-derived supply whenever both are present (e.g. flashing firmware on a board that's already powered — a completely normal case for a board that'll be reflashed constantly during bring-up). So there's a small dedicated path: **VBUS → a 3.3V LDO (reusing AMS1117-3.3, already the part chosen for the main 3.3V rail) → a Schottky diode → the shared 3.3V rail.** The main regulation's 3.3V LDO output connects to that same shared rail directly, with no diode in series — asymmetric on purpose. If it had a diode too, the main rail would drop by the diode's forward voltage during *every* normal battery-powered run, not just when USB is present. With the diode only on the USB leg: whenever battery power is present, the main regulator's tightly-regulated output sits at or above the USB-LDO's output, which naturally reverse-biases (blocks) the USB diode with no active switching needed — no backfeed, full clean 3.3V preserved. When only USB is present (no battery), the diode conducts and the USB-LDO supplies the rail. This lets the board be flashed/debugged from USB alone on the bench with no battery connected, without any risk when both sources are live at once. Sized for logic-only loads (MCU + peripherals) — not intended to power the SA818S PA at TX current.

### 4.8 RF power sensing — forward power only, dual-path (ADC + comparator)

Simple diode detector on the antenna line, forward power only (no reflected/SWR) for this revision. The detector's output feeds two parallel paths rather than just an ADC:

- **ADC1 (IO4)** — actual power-level readback, for calibration/verification of "variable TX power" (otherwise the PWM-based power setting is open-loop and unverified). Deliberately ADC1, not ADC2 — ADC2 is unreliable while WiFi is active.
- **Comparator (e.g. LM393-class) → digital GPIO (IO41)** — a fast, software-independent fault flag ("near-zero power leaving, probably an open/damaged antenna"). Reference threshold set via a resistor divider off 3.3V (exact values TBD during detector circuit design). Added specifically because a comparator's digital HIGH/LOW is far more noise-immune near an active RF PA than trying to resolve a precise analog level — the ADC path stays for the number you actually want (power level), the comparator adds a robust independent trip you can trust without relying on ADC averaging/timing.

This is the same pattern used in commercial RF gear: fast comparator trip for fault/protection, slower ADC readout for the calibrated value.

### 4.9 Status indication

**Hardware debug LEDs** (footprints reserved, DNP by default — populate only for bench bring-up) at key power-section nodes: battery-protected output (`PWR_18650`), `+12V` (post-fuse), `PWR_IN_SELECT` (post-ORing), `5V_BUCK_OUT`, `3V3_BUCK_OUT`. **Exception:** the final `+3.3V` rail LED is hard-populated (always on) — it's the one purely passive "logic power present" indicator, since no MCU-driven LED can report anything if 3.3V never came up in the first place.

**MCU-driven status LEDs:** a dedicated TX-active LED (own GPIO, not folded into a color code — RF safety/awareness deserves an unambiguous indicator), a dedicated heartbeat LED (brief periodic blink, not solid, to confirm firmware is running without a continuous-draw cost), and one RGB/WS2812 LED encoding GPS search/lock, fault conditions (battery critical, antenna fault, SD error), and found-log mode active (WiFi/BLE on).

### 4.10 TX safety — watchdog + independent hardware PTT timeout

ESP32-S3's internal watchdog timers cover general firmware-hang protection — no separate general-purpose supervisor IC needed. **But** a stuck-transmitter fault (firmware hang *or* a logic bug that keeps PTT asserted while the CPU is otherwise fine) needs its own independent layer, since an MCU reset doesn't guarantee the PTT line de-asserts, and a pure software watchdog can't catch "firmware is running but wrongly still keying."

- **PTT line defaults to released** whenever its driving GPIO is undriven/in reset (pull resistor sized accordingly), so a reset transient can't leave the radio keyed.
- **Independent hardware TX-timeout:** a 74HC123 dual retriggerable monostable (RC-timed for a **2-minute hard ceiling**) gates the PTT line in series with the MCU's own PTT-intent GPIO. The MCU must periodically retrigger it during a legitimate transmission; if it stops (hang or logic bug), the 74HC123's timeout forces PTT release after 2 minutes regardless of MCU state — independent of whether the MCU's own watchdog even fires.

### 4.11 Frequency / tone plan

Arbitrary **frequency register** (not a fixed channel list) — matches the "flexible and dynamic" goal. CTCSS/DCS tone supported by default, including **randomizing tone within a valid range** between transmission cycles. Both are SA818S UART commands (`AT+DMOSETGROUP` et al.) — zero hardware impact, pure firmware/config-data-model concerns (same EEPROM config bucket as §4.6).

### 4.12 LoRa / remote telemetry — skipped

Considered for knowing the fox is still alive without physically visiting it, but skipped entirely for this revision (no reserved footprint) — not useful without a receiver/backbone plan (e.g. Meshtastic) already in place, which doesn't exist yet. Revisit if that infrastructure materializes.

### 4.13 Board / enclosure

No size constraint for this revision — bare board, no enclosure, decided during layout. GPS antenna is external via SMA (§4.3); SA818S antenna presumably SMA as well (TBD at layout).

## Downstream Loads (planned, not yet on this board)

| Device | Rail | Typical | Peak |
|---|---|---|---|
| SA818S (2m RF module) | 5V | ~60mA RX | ~750mA TX |
| ESP32-S3-WROOM-1-N8R8 | 3.3V | ~20–80mA | ~300–400mA (radio TX burst) |
| u-blox MAX-M10S GPS | 3.3V | ~25–45mA | ~50–70mA (acquisition) |

Controller-section peripherals (RTC, GPS, EEPROM, SD card, audio DAC, status LEDs, 74HC123) are all low-current (µA–low-mA class) individually; a full current budget for the controller section is still TBD once that schematic exists and real component picks are finalized.

## Open Items / Next Steps

- Confirm GNDPWR solder-jumper flags are wired to the intended nets (not accidentally re-merging GND/GNDREF) — likely a KiCad global GND power-symbol mixup if it recurs.
- Populate downstream loads (SA818S, ESP32-S3, GPS) and re-verify rail current budgets against real hardware.
- First board bring-up: isolate each stage via the power-section jumpers, verify independently, then re-bridge for full-system test.
- Begin schematic capture for the §4 controller/peripheral platform — none of it exists in `.kicad_sch` yet, this section is planning-only.
- Once the controller section has real current draws, re-verify the existing 5V/2A and 3.3V/1A regulation budget still covers it.
