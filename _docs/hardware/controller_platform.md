# Controller & Peripheral Platform

Design decisions for the controller section. **Power sequencing, Timekeeping, status
indication (I2C GPIO expander + WS2812 status LED), and the EEPROM (storage) are now
implemented in the schematic** (`power.kicad_sch`'s soft-latch stage; the `Peripherals`
sheet's PCF8563 RTC, PCA9555 expander, and CAT24C32YI-GT3 EEPROM circuits; the WS2812 LED
on `mcu-esp32.kicad_sch`; the SD card interface, also on `mcu-esp32.kicad_sch`); GPS,
audio/PTT path, RF power sensing, and TX-safety timeout are still planning-only. See [Open Items](open_items.md) for what's left. Target
capabilities:
audio/voice TX, CW ID keying, variable-duration/interval blipping, variable TX power, APRS
TX with GPS (dithered position optional), randomized CTCSS tone, frequency hopping, a
WiFi/BLE "found" log, RTC-scheduled precise-interval TX, and schedulable mode changes
(e.g. more frequent TX the longer a fox goes unfound).

## Power sequencing — discrete soft-latch (implemented)

MAX17320 protects/fuel-gauges the battery but doesn't gate system power — that's a separate circuit. A P-FET soft-latch (Q5, AO3407A — picked over AO3401A for its ±20V Vgs rating, comfortably covering VRAW's 7–14.4V swing with no gate-clamp zener needed) sits between VRAW and Power Regulation's input, gated by the shared wake node:

- **Wake sources are wire-ORed** (not literally diode-ORed — see below) onto one active-low node, pulled up to VRAW through R29 (1MΩ, sized to keep standby bleed in the single-digit µA range across VRAW's full voltage span rather than something that would meaningfully shorten shelf life): the power button (SW3), the RTC interrupt chain, and the MCU's own latch-hold output. Any of them pulling the node toward GND turns Q5 on.
- **MCU self-latch:** GPIO48 (open-drain intent, 3.3V logic) drives Q6 (BSS138) as a voltage-isolating buffer — GPIO48 can never be exposed to VRAW's up-to-14.4V swing, only ever sees 0–3.3V on its own side. Q6 pulls the wake node low to hold power on; releasing it (firmware shutdown housekeeping) lets R29 pull the node back to VRAW and the system goes fully dark — true zero-current off, not MCU sleep.
- **Hold-to-power-off detection — implemented.** SW3 needed its own isolated read of "is the button physically being held," independent of whatever else is currently asserting the shared wake node (RTC INT, or the MCU's own self-latch above) — reading the wake node directly would have made "button held" indistinguishable from "system is just normally running." Solved the same way as the RTC INT path: SW3's pin 2 now lands on its own node (`POWER_PB_SIGNAL`), pulled up to VRAW through R45 (1MΩ, same standby-current reasoning as R29) instead of tying directly to the wake node. D5 (BAS116LT1G, anode on the wake-node side / `PWR_LATCH`, cathode on `POWER_PB_SIGNAL`) diode-ORs SW3's press back into the wake node so it still asserts power-on as before, while blocking the reverse path — so nothing else asserting the wake node can pull `POWER_PB_SIGNAL` down. That isolated signal crosses the hierarchy (`POWER_PB_SIGNAL` → `MCU_IN_PWR_BTN`) to a BSS138 stage (Q9) on the `mcu-esp32` sheet, which level-shifts it into GPIO38 (R44, 10k pull-up) for firmware to poll. Verified: idle → GPIO38 low; SW3 held → GPIO38 high (active-high on this pin specifically, opposite the passive-pull-up-active-low convention used elsewhere — worth flagging clearly in firmware).
- **Still open:** the firmware itself (polling GPIO38, timing the hold, releasing GPIO48) hasn't been written — schematic only exposes the signal. And a **firmware-independent hardware force-off** is still unbuilt — see [Open Items](open_items.md) for the leading option (spare half of the TX-safety 74HC123).
- **RTC wake path:** PCF8563's `INT` is open-drain **active-low**, not active-high, and its own absolute max voltage rating (6.5V) is well under VRAW's range — so it can't be wired to the wake node directly either. The actual chain is `INT` → Q8 (AO3407A, a small 3.3V-domain inverter so the *sense* comes out right — INT low asserts, not releases) → Q7 (BSS138, the same voltage-isolating role as Q6) → wake node. R30 (10k) is INT's own pull-up; R31 (100k) holds Q7's gate at a defined low/off state whenever Q8 isn't actively driving it. This is more circuitry than "diode-ORed" implies, but the earlier plan's instinct was right — INT genuinely can't touch the high-voltage node without protection.
- Bench-test provision: JP24 (`usbc-programming-uart.kicad_sch`) can exclude USB from both TPS2121 rail muxes, so the whole sleep/wake cycle is testable on the bench with a debug USB cable still attached (see [regulation.md](regulation.md) for why gating downstream of the muxes instead was rejected — it would leave the bucks/LDOs always powered, defeating the standby-current point).

## Timekeeping — PCF8563 RTC (implemented)

**PCF8563** (I2C, external 32.768kHz crystal — NDK NX3215SA-32.768K-STD-MUA-9, 9pF, matching the datasheet's CL=8pF characterization condition better than a 12.5pF part would), not DS3231. DS3231's integrated TCXO gives better standalone accuracy (±2ppm) and needs no external crystal, but costs ~15-20x more (~$9-10.50 vs. ~$0.50 at JLCPCB) for a benefit this design doesn't need — accuracy is maintained instead by periodically disciplining the RTC from GPS whenever it has a fix, and drift over a single unattended event is forgiving either way. PCF8563 has an integrated OSCO-side load capacitor, so no discrete load-cap pair is needed — just an optional DNP trim cap (C43) on OSCI if fine-tuning ever proves necessary.

Backup: **coin cell** (CR2032, BT3), diode-ORed at VDD with the main 3.3V rail (D2/D3, Nexperia BAS116 — genuinely low-leakage, ~3pA typical, verified against its own datasheet rather than assumed, since ordinary diode reverse leakage can be comparable to or exceed the RTC's own nanoamp-class backup draw). No supercap option for the RTC specifically — a supercap can't hold time across the months/years this board may sit idle between events (self-discharges in days-to-weeks), where a coin cell holds ~10 years. Losing RTC time after long storage is an accepted, low-consequence tradeoff (resolved for free by GPS resync at next power-up).

**Firmware note:** `CLKOUT` defaults to *enabled* (32.768kHz) at power-on per the register reset table (`CLKOUT_control`, address `0Dh`, bit `FE`) — unused in this design (left unconnected), but firmware must explicitly disable it (`FE=0`) or the RTC's backup current roughly doubles (500–750nA → 950–1700nA per the datasheet) for a pin nothing listens to.

I2C bus (`PERIPH_SDA`/`PERIPH_SCL`, shared with the MAX17320 fuel gauge and — once built — the EEPROM) has its one pull-up pair (R32/R33, 10k to 3.3V) on the `mcu-esp32` sheet, not duplicated per device.

## GPS — u-blox MAX-M10S

UART+I2C, PPS output, external active antenna via **SMA connector** (not an integrated patch). Backup power kept **separate from the RTC's coin cell** — its own backup domain with both a coin-cell holder and a supercap footprint, user populates either/neither/both at assembly (same low-leakage diode-OR pattern as the RTC), feeding VBACKUP so ephemeris/RTC survive short sleeps for a fast warm-start instead of a full cold-start reacquisition.

## MCU — ESP32-S3-WROOM-1-N8R8

Module (not bare chip) — this board already has one RF section to get right (SA818S); a second self-laid-out antenna-matching problem on the MCU's WiFi/BLE radio isn't worth it for this revision. N8R8 (8MB flash + 8MB PSRAM) — PSRAM matters more than extra flash for the concurrent WiFi + audio buffering + SD card workload. **BLE enabled** alongside WiFi — free with the same radio, gives a lower-power/faster-handshake alternative to the WiFi captive portal for the "found" log.

Full pin-by-pin plan: [esp32s3_pinout.md](esp32s3_pinout.md).

**JTAG comes free over the native USB-C connector** — ESP32-S3's native USB includes a built-in USB Serial/JTAG peripheral (CDC serial + full OpenOCD-compatible debug access, simultaneously, no extra pins). No separate JTAG header needed. BOOT/RESET buttons are still included regardless as a manual fallback.

**Two USB connectors, not a hub chip: native USB direct to the module, plus a separate USB-UART bridge on its own port.** Considered mirroring Espressif's newer DevKitC-1 reference design (single USB-C → onboard USB hub chip → both a USB-UART bridge and the module's native USB), but that pattern earns its keep on a retail dev board where connector count matters to strangers unfamiliar with the board. Here it just adds a hub IC, its own layout/power budget, and another thing that can fail, for no benefit this project needs. Two small connectors is cheaper and simpler:

- **Native USB-C** — direct to IO19/IO20 (USB_D-/D+), as before: JTAG, native USB CDC, and esptool flashing.
- **USB-UART bridge (CP2102N-class), own connector** — solves two real annoyances with native-USB-only boards during active firmware iteration: (1) the native USB CDC device re-enumerates on every reset/flash cycle, which drops/delays the serial monitor and can eat early boot log lines; a UART bridge's virtual COM port stays enumerated across ESP32-S3 resets. (2) auto-reset-into-bootloader over native USB has known version-dependent quirks (esptool/OS-driver dependent) — a real bridge's DTR/RTS auto-reset into EN/IO0 (the standard two-transistor circuit) is reliable and well-established. Manual BOOT/RESET buttons remain as the ultimate fallback either way (e.g. no bridge cable plugged in).

## Audio / PTT path (shared by onboard SA818S and external HT)

The SA818S has **no digital baseband input** — its UART is control-only (frequency, squelch, volume, CTCSS/DCS). Voice, CW tone, and AFSK for APRS all have to arrive as analog audio on its MIC pin, so an audio DAC (I2S DAC → RC filter, or PWM+filter) is required regardless of "digital vs. analog" framing.

This is one shared subsystem serving two destinations, not two separate designs: the same DAC output and PTT-keying circuit (optocoupler) route to *both* the onboard SA818S footprint (populated later for eval, unlikely at initial fab per original scope) *and* a 3.5mm jack for an external HT, switchable/jumpered between the two.

## Storage — SD card + EEPROM

- **SD card: SPI mode — implemented.** Not SDIO. No audio *recording* planned, and audio *playback* (voice IDs, custom clips) is a light, bufferable workload (~32KB/s for 16kHz/16-bit mono) well within SPI-mode throughput — SDIO's speed and extra GPIO cost isn't needed.
- **Connector: Molex 472192001** (LCSC `C164170`, ref `Card1` on `mcu-esp32.kicad_sch`) — flip-top/hinged microSD socket, 8-pin SMD, no card-detect pin, 1.9mm height above board, ~13.7k units in stock. Hinged style over push-push specifically requested — easier to swap cards during bench bring-up than a push-push socket's insert-to-eject action.
- **Wiring**: SPI2 (`FSPI`) peripheral, matching this project's IO-matrix pin plan — IO10 (`FSPICS0`) → `CD/DAT3` (CS), IO11 (`FSPID`) → `CMD` (MOSI/DI), IO12 (`FSPICLK`) → `CLK` (SCLK), IO13 (`FSPIQ`) → `DAT0` (MISO/DO). `DAT1`/`DAT2` correctly left unconnected (SD-native-mode-only signals, unused in SPI mode). Shield/mounting tabs (pins 9-12, renamed `SHLD` in the symbol) tied to GND for EMI shielding — the part is explicitly a *shielded* connector, so these are a real shield-ground connection, not inert mechanical pads. Local 0.1µF decoupling cap (C53) on VDD.
- **Open item**: no pull-up on the CS (`CD/DAT3`) line yet. Espressif's own SD pull-up requirements doc specifies 10kΩ on CMD/DAT0-3 for SPI mode — matches every other control-line pull-up already used on this board (I2C, PCA9555 INT, auto-reset network, RTC INT all use 10kΩ). Not strictly required since this SPI bus isn't shared with any other device, but recommended before final layout.
- **Config storage: I2C EEPROM — implemented.** onsemi **CAT24C32YI-GT3** (32Kbit, TSSOP-8, LCSC `C94264`), address pins grounded → `0x50`, placed on the `Peripherals` sheet (U16) with its own local 0.1µF decoupling cap. Picked over the originally-named Microchip AT24C32D-SSHM-T during layout — both are genuine drop-ins on the industry-standard 24Cxx pinout (see `ic_inventory.md`), this was just the one sourced. WP is wired to a solder jumper (open by default, floating → internal pull-down → write-protect disabled; bridge it to +3.3V to hardware-lock the EEPROM) rather than hard-tied, matching this board's bench-testable jumper-isolation convention. Not FRAM — FRAM (FM24C/MB85RC families) isn't available in JLCPCB's catalog, and EEPROM's ~1M write-cycle life is a non-issue for settings that change occasionally (schedule, frequency, TX power, tone), not continuously. Lives on the same I2C bus as the RTC/fuel-gauge, independent of the SD card, so critical operating parameters survive a missing/corrupt card.

## Onboard charging — none; USB powers logic only, via an isolated path

**No battery charging.** Investigated: TI **BQ25792** (I2C-configurable 1-4S buck-boost charger, in stock at JLCPCB) can charge the 2S1P pack directly from 5V USB without PD negotiation (internally boosts as needed) — a real, buildable option, not ruled out for cost/feasibility reasons. Decided against it for this revision anyway; charging stays external/separate as originally designed.

**USB VBUS is not left electrically dangling, though** — every USB connection carries 5V on VBUS regardless of intent, and if that were wired straight into the 3.3V rail or VRAW it would fight the battery-derived supply whenever both are present (e.g. flashing firmware on a board that's already powered — a completely normal case for a board that'll be reflashed constantly during bring-up). So there's a small dedicated path: **VBUS → a 3.3V LDO (TI LM1117IMPX-3.3/NOPB, the part now used for both the main and USB-side 3.3V rails — swapped from the originally-placed AMS1117-3.3, see `ic_inventory.md`) → a Schottky diode → the shared 3.3V rail.** The main regulation's 3.3V LDO output connects to that same shared rail directly, with no diode in series — asymmetric on purpose. If it had a diode too, the main rail would drop by the diode's forward voltage during *every* normal battery-powered run, not just when USB is present. With the diode only on the USB leg: whenever battery power is present, the main regulator's tightly-regulated output sits at or above the USB-LDO's output, which naturally reverse-biases (blocks) the USB diode with no active switching needed — no backfeed, full clean 3.3V preserved. When only USB is present (no battery), the diode conducts and the USB-LDO supplies the rail. This lets the board be flashed/debugged from USB alone on the bench with no battery connected, without any risk when both sources are live at once. Sized for logic-only loads (MCU + peripherals) — not intended to power the SA818S PA at TX current.

## RF power sensing — forward power only, dual-path (ADC + comparator)

Simple diode detector on the antenna line, forward power only (no reflected/SWR) for this revision. The detector's output feeds two parallel paths rather than just an ADC:

- **ADC1 (IO4)** — actual power-level readback, for calibration/verification of "variable TX power" (otherwise the PWM-based power setting is open-loop and unverified). Deliberately ADC1, not ADC2 — ADC2 is unreliable while WiFi is active.
- **Comparator (e.g. LM393-class) → digital GPIO (IO41)** — a fast, software-independent fault flag ("near-zero power leaving, probably an open/damaged antenna"). Reference threshold set via a resistor divider off 3.3V (exact values TBD during detector circuit design). Added specifically because a comparator's digital HIGH/LOW is far more noise-immune near an active RF PA than trying to resolve a precise analog level — the ADC path stays for the number you actually want (power level), the comparator adds a robust independent trip you can trust without relying on ADC averaging/timing.

This is the same pattern used in commercial RF gear: fast comparator trip for fault/protection, slower ADC readout for the calibrated value.

## Status indication

**Hardware debug LEDs** (footprints reserved, DNP by default — populate only for bench bring-up) at key power-section nodes: battery-protected output (`PWR_18650`), `+12V` (post-fuse), `PWR_IN_SELECT` (post-ORing), `5V_BUCK_OUT`, `3V3_BUCK_OUT`. **Exception:** the final `+3.3V` rail LED is hard-populated (always on) — it's the one purely passive "logic power present" indicator, since no MCU-driven LED can report anything if 3.3V never came up in the first place.

**MCU-driven status LEDs:** a dedicated TX-active LED (not folded into a color code — RF safety/awareness deserves an unambiguous indicator), a dedicated heartbeat LED (brief periodic blink, not solid, to confirm firmware is running without a continuous-draw cost), and one RGB/WS2812 LED encoding GPS search/lock, fault conditions (battery critical, antenna fault, SD error), and found-log mode active (WiFi/BLE on).

**I2C GPIO expander — implemented, but as a general-purpose expansion block, not a dedicated LED driver.** TI **PCA9555PWR** (U15, TSSOP-24, address pins grounded → `0x20`) is placed on the `Peripherals` sheet with its own `INT` output wired to the MCU (IO39, see `esp32s3_pinout.md`) so firmware can be notified on any input change instead of polling. What's actually behind it right now: 4 general-purpose LEDs (green/red/blue/yellow, real Vf-matched series resistors, see below) and 8 general-purpose switch inputs (a 4-position onboard DIP switch + 4 onboard pushbuttons + 4 external JST breakouts with pull-ups) — none of the 8 output or 8 input lines are wired to a specific "TX-active"/"heartbeat" role in the schematic; that assignment is now a firmware/config decision rather than a hardware one. This is what freed IO3 back to spare in `esp32s3_pinout.md` (IO39 itself is used by the expander's `INT`, not freed).

**RGB/WS2812 status LED — implemented.** D4, Worldsemi **WS2812B-B/W** (SMD5050-4P, LCSC `C114586`), on its own native GPIO (IO40/pin 33) rather than behind the expander — it needs a real serial protocol with strict bit timing (RMT peripheral), which a static-register expander can't produce. `DIN` wired to IO40; `DOUT` explicitly no-connect flagged since it's the only LED in the chain (safe to leave floating per the part's own datasheet — nothing downstream to feed). Local 0.1µF bypass cap (C52) at VDD/VSS.

## TX safety — watchdog + independent hardware PTT timeout

ESP32-S3's internal watchdog timers cover general firmware-hang protection — no separate general-purpose supervisor IC needed. **But** a stuck-transmitter fault (firmware hang *or* a logic bug that keeps PTT asserted while the CPU is otherwise fine) needs its own independent layer, since an MCU reset doesn't guarantee the PTT line de-asserts, and a pure software watchdog can't catch "firmware is running but wrongly still keying."

- **PTT line defaults to released** whenever its driving GPIO is undriven/in reset (pull resistor sized accordingly), so a reset transient can't leave the radio keyed.
- **Independent hardware TX-timeout:** a 74HC123 dual retriggerable monostable (RC-timed for a **2-minute hard ceiling**) gates the PTT line in series with the MCU's own PTT-intent GPIO. The MCU must periodically retrigger it during a legitimate transmission; if it stops (hang or logic bug), the 74HC123's timeout forces PTT release after 2 minutes regardless of MCU state — independent of whether the MCU's own watchdog even fires.

## Frequency / tone plan

Arbitrary **frequency register** (not a fixed channel list) — matches the "flexible and dynamic" goal. CTCSS/DCS tone supported by default, including **randomizing tone within a valid range** between transmission cycles. Both are SA818S UART commands (`AT+DMOSETGROUP` et al.) — zero hardware impact, pure firmware/config-data-model concerns (same EEPROM config bucket as storage, above).

## LoRa / remote telemetry — skipped

Considered for knowing the fox is still alive without physically visiting it, but skipped entirely for this revision (no reserved footprint) — not useful without a receiver/backbone plan (e.g. Meshtastic) already in place, which doesn't exist yet. Revisit if that infrastructure materializes.

## Board / enclosure

No size constraint for this revision — bare board, no enclosure, decided during layout. GPS antenna is external via SMA (see GPS section above); SA818S antenna presumably SMA as well (TBD at layout).
