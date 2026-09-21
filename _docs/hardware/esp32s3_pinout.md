# ESP32-S3-WROOM-1-N8R8 — Pinout & Interconnect Plan

Source: Espressif *ESP32-S3-WROOM-1 & WROOM-1U* datasheet v1.3 (pulled directly from the
module's LCSC-hosted datasheet PDF, not from memory — pin numbers/alt-functions below are
transcribed from the datasheet's own pin table).

**Module variant note:** our pick is the **R8** PSRAM variant (8MB Octal SPI PSRAM,
integrated in-module). On R8/R16V variants, **pins 28–30 (IO35/IO36/IO37) are internally
wired to that PSRAM and are not available for any other use** — they don't appear as
routable nets in the interconnect table below. This is easy to miss and would have been a
real problem if assigned to something.

## 1. Full reference pin table (as datasheet)

| Pin # | Name | Type | Datasheet alt-functions |
|---|---|---|---|
| 1 | GND | P | Ground |
| 2 | 3V3 | P | Supply |
| 3 | EN | I | Chip enable/reset (active high). Must not float. |
| 4 | IO4 | I/O/T | RTC_GPIO4, GPIO4, TOUCH4, ADC1_CH3 |
| 5 | IO5 | I/O/T | RTC_GPIO5, GPIO5, TOUCH5, ADC1_CH4 |
| 6 | IO6 | I/O/T | RTC_GPIO6, GPIO6, TOUCH6, ADC1_CH5 |
| 7 | IO7 | I/O/T | RTC_GPIO7, GPIO7, TOUCH7, ADC1_CH6 |
| 8 | IO15 | I/O/T | RTC_GPIO15, GPIO15, U0RTS, ADC2_CH4, XTAL_32K_P |
| 9 | IO16 | I/O/T | RTC_GPIO16, GPIO16, U0CTS, ADC2_CH5, XTAL_32K_N |
| 10 | IO17 | I/O/T | RTC_GPIO17, GPIO17, U1TXD, ADC2_CH6 |
| 11 | IO18 | I/O/T | RTC_GPIO18, GPIO18, U1RXD, ADC2_CH7, CLK_OUT3 |
| 12 | IO8 | I/O/T | RTC_GPIO8, GPIO8, TOUCH8, ADC1_CH7, SUBSPICS1 |
| 13 | IO19 | I/O/T | RTC_GPIO19, GPIO19, U1RTS, ADC2_CH8, CLK_OUT2, **USB_D-** |
| 14 | IO20 | I/O/T | RTC_GPIO20, GPIO20, U1CTS, ADC2_CH9, CLK_OUT1, **USB_D+** |
| 15 | IO3 | I/O/T | RTC_GPIO3, GPIO3, TOUCH3, ADC1_CH2 — **strapping: JTAG source** (default floating) |
| 16 | IO46 | I/O/T | GPIO46 — **strapping: boot mode / ROM log** (default pull-down) |
| 17 | IO9 | I/O/T | RTC_GPIO9, GPIO9, TOUCH9, ADC1_CH8, FSPIHD, SUBSPIHD |
| 18 | IO10 | I/O/T | RTC_GPIO10, GPIO10, TOUCH10, ADC1_CH9, FSPICS0, FSPIIO4, SUBSPICS0 |
| 19 | IO11 | I/O/T | RTC_GPIO11, GPIO11, TOUCH11, ADC2_CH0, FSPID, FSPIIO5, SUBSPID |
| 20 | IO12 | I/O/T | RTC_GPIO12, GPIO12, TOUCH12, ADC2_CH1, FSPICLK, FSPIIO6, SUBSPICLK |
| 21 | IO13 | I/O/T | RTC_GPIO13, GPIO13, TOUCH13, ADC2_CH2, FSPIQ, FSPIIO7, SUBSPIQ |
| 22 | IO14 | I/O/T | RTC_GPIO14, GPIO14, TOUCH14, ADC2_CH3, FSPIWP, FSPIDQS, SUBSPIWP |
| 23 | IO21 | I/O/T | RTC_GPIO21, GPIO21 |
| 24 | IO47 | I/O/T | SPICLK_P_DIFF, GPIO47, SUBSPICLK_P_DIFF |
| 25 | IO48 | I/O/T | SPICLK_N_DIFF, GPIO48, SUBSPICLK_N_DIFF |
| 26 | IO45 | I/O/T | GPIO45 — **strapping: VDD_SPI voltage** (default pull-down) |
| 27 | IO0 | I/O/T | RTC_GPIO0, GPIO0 — **strapping: boot mode** (default pull-up) |
| 28 | IO35 | I/O/T | **NOT AVAILABLE — internal Octal PSRAM (R8 module)** |
| 29 | IO36 | I/O/T | **NOT AVAILABLE — internal Octal PSRAM (R8 module)** |
| 30 | IO37 | I/O/T | **NOT AVAILABLE — internal Octal PSRAM (R8 module)** |
| 31 | IO38 | I/O/T | GPIO38, FSPIWP, SUBSPIWP |
| 32 | IO39 | I/O/T | MTCK, GPIO39, CLK_OUT3, SUBSPICS1 |
| 33 | IO40 | I/O/T | MTDO, GPIO40, CLK_OUT2 |
| 34 | IO41 | I/O/T | MTDI, GPIO41, CLK_OUT1 |
| 35 | IO42 | I/O/T | MTMS, GPIO42 |
| 36 | RXD0 | I/O/T | U0RXD, GPIO44, CLK_OUT2 |
| 37 | TXD0 | I/O/T | U0TXD, GPIO43, CLK_OUT1 |
| 38 | IO2 | I/O/T | RTC_GPIO2, GPIO2, TOUCH2, ADC1_CH1 |
| 39 | IO1 | I/O/T | RTC_GPIO1, GPIO1, TOUCH1, ADC1_CH0 |
| 40 | GND | P | Ground |
| 41 | EPAD | P | Ground (exposed thermal pad) |

Strapping pins (GPIO0, GPIO3, GPIO45, GPIO46) latch their value at reset and behave as
normal GPIO afterward — avoid strong external pulls on these unless deliberately changing
boot behavior.

## 2. Proposed interconnect assignment

Note on flexibility: except for **USB D+/D-** (hard-wired to IO19/IO20 on this chip) and the
UART0/strapping pins noted above, nearly everything below is GPIO-matrix-assignable —
these are *my proposed* pin choices for layout tidiness (grouping related signals,
matching datasheet alt-function labels where convenient, e.g. SD card on the FSPI-labeled
pins), not silicon requirements. Easy to reshuffle.

**Wired column (far right)** — cross-checked directly against the live `.kicad_sch` files
(pin-level net trace, not assumed from this table), not just this doc's own stated intent.

| Symbol | Meaning |
|---|---|
| ✅ | Wired — pin already lands on a real net in the schematic today |
| *(blank)* | Planned — signal/subsystem doesn't exist in the schematic yet; this doc's assignment is a reservation, not a fact about current hardware |
| ❓ | Not yet assigned — genuinely spare, no purpose decided |
| ❌ | Do not connect — strapping pin or internally-reserved pin that must stay floating/unavailable, not a candidate for wiring |

| Pin # | Signal | Connects to | Notes | Wired |
|---|---|---|---|---|
| 1 | GND | Ground plane | | ✅ |
| 2 | 3V3 | `+3.3V` rail | | ✅ |
| 3 | EN | Reset circuit: 10kΩ pull-up to 3V3 + 1µF to GND (power-on delay) + manual RESET button to GND + auto-reset transistor from the USB-UART bridge's DTR line (§ MCU in controller_platform.md) | Standard practice | ✅ |
| 4 | IO4 | Forward-power detector output (ADC, level readback) | **ADC1** channel — deliberately not ADC2; ADC2 is unreliable while WiFi is active. Paired with a comparator fault-flag on IO41 (pin 34) — see §4.8 in README for the dual-path design | |
| 5 | IO5 | GPS PPS input | Interrupt-capable, precise edge capture. `GPS_1PPS_IN`, confirmed wired to `U18` TIMEPULSE. | ✅ |
| 6 | IO6 | SA818S UART — MCU TX → SA818 RXD | UART2 (software-assigned). Verified correct crossover against the live schematic. | ✅ |
| 7 | IO7 | SA818S UART — MCU RX ← SA818 TXD | UART2 (software-assigned). Verified correct crossover against the live schematic. | ✅ |
| 8 | IO15 | *Spare (freed)* | Was planned as one leg of a PTT AND-gate (with a 74HC123 timeout output on IO16) — that scheme wasn't what got built. See pin 9: PTT ended up as a single direct GPIO into a FET, no second GPIO needed for now. Genuinely spare again until the TX-safety-timeout stage is actually designed. | ❓ |
| 9 | IO16 | SA818S/HT PTT-intent (MCU output) | Drives `Q10` (BSS138) gate directly — a simple low-side FET switch, not the AND-gate/74HC123 scheme originally planned here. `Q10`'s drain feeds the switched-jack circuit (`J11`) that auto-routes PTT to either the onboard SA818S or an external HT — see `audio_ptt_path.md` and `sa818_pinout.md`. TX-safety timeout (74HC123) is still a separate, not-yet-built stage; when added, it'll gate somewhere in this same path rather than needing a second GPIO here. | ✅ |
| 10 | IO17 (U1TXD) | GPS module RXD | UART1, matches datasheet's own U1TXD label. `MCU_GPS_TXD`, verified correct crossover into `U18` RXD. | ✅ |
| 11 | IO18 (U1RXD) | GPS module TXD | UART1. `MCU_GPS_RXD`, verified correct crossover from `U18` TXD. | ✅ |
| 12 | IO8 | I2C SDA | Shared bus: PCF8563 RTC (`0x51`), MAX17320 fuel gauge (`0x36` main / `0x0B` NVM), CAT24C32YI-GT3 EEPROM (`0x50`), and PCA9555PWR I2C GPIO expander (`0x20`) — all four placed and wired | ✅ |
| 13 | IO19 | **USB_D-** | Native USB, fixed pin | ✅ |
| 14 | IO20 | **USB_D+** | Native USB, fixed pin | ✅ |
| 15 | IO3 | I2S DAC `XSMT` mute control | Was TX-active LED, then freed (moved to the I2C GPIO expander), now spent on the audio circuit's XSMT net (`I2C_MUTE_XSMT`) — the board's last genuinely-spare pin, now used up. **Strapping pin (JTAG source, default floating)** — the audio sheet has a 10kΩ pull-up to `+3.3V` on this net (R46); worth double-checking against Espressif's own strapping table that a weak pull-up in this direction doesn't select a non-default JTAG-source state at boot, before this is relied on. | ✅ |
| 16 | IO46 | **Reserved / NC** | Strapping pin (boot mode + ROM log), leave floating for default boot behavior | ❌ |
| 17 | IO9 | I2C SCL | Shared bus, see pin 12 | ✅ |
| 18 | IO10 (FSPICS0) | SD card CS (`Card1` pin 2, `CD/DAT3`) | No CS pull-up yet — see `open_items.md`, recommend 10kΩ to `+3.3V` per Espressif's SD pull-up spec | ✅ |
| 19 | IO11 (FSPID) | SD card MOSI/DI (`Card1` pin 3, `CMD`) | `FSPID` = data-out-from-host per ESP32's SPI-flash-derived naming (D=MOSI, Q=MISO) — not obvious from the datasheet's own pin table | ✅ |
| 20 | IO12 (FSPICLK) | SD card SCLK (`Card1` pin 5, `CLK`) | | ✅ |
| 21 | IO13 (FSPIQ) | SD card MISO/DO (`Card1` pin 7, `DAT0`) | `DAT1`/`DAT2` (native-mode-only signals) correctly left unconnected — SPI mode doesn't use them | ✅ |
| 22 | IO14 | I2S BCLK → audio DAC | DAC is `U17` (PCM5102A), placed in `audio.kicad_sch`. `I2S_BCK`, confirmed wired. | ✅ |
| 23 | IO21 | I2S LRCLK/WS → audio DAC | `I2S_LRCK`, confirmed wired. | ✅ |
| 24 | IO47 | I2S DOUT → audio DAC | `I2S_DIN`, confirmed wired. No MCLK pin needed — PCM5102A runs its internal PLL off `SCK` tied low (see judgment call #2). | ✅ |
| 25 | IO48 | Power-latch hold (MCU output, open-drain) | Diode-ORed with button + PCF8563 INT at the soft-latch enable node (§4.1) | ✅ |
| 26 | IO45 | **Reserved / NC** | Strapping pin (VDD_SPI voltage), leave floating for default (3.3V) behavior | ❌ |
| 27 | IO0 | BOOT button (pull to GND) + auto-reset transistor from the USB-UART bridge's RTS line | Strapping pin (boot mode). Manual BOOT+RESET buttons stay as the fallback when the bridge isn't plugged in; the bridge's RTS/DTR pair drives the standard two-transistor auto-reset circuit for one-command `esptool`/PlatformIO uploads | ✅ |
| 28 | IO35 | — | Not available (internal PSRAM) | ❌ |
| 29 | IO36 | — | Not available (internal PSRAM) | ❌ |
| 30 | IO37 | — | Not available (internal PSRAM) | ❌ |
| 31 | IO38 | Hold-to-power-off detect (input, active-high) | Not a second switch pole — SW3 stays a plain SPST button. Its own node (`POWER_PB_SIGNAL`) is isolated from the shared wake node via R45 (1MΩ pull-up) + D5 (diode-OR back into `PWR_LATCH`), then level-shifted through Q9 (BSS138) into this pin (R44, 10k pull-up) — see § Power sequencing in controller_platform.md. Firmware still needs to poll and time the hold; a firmware-independent hardware force-off is still open (see `open_items.md`) | ✅ |
| 32 | IO39 (MTCK) | PCA9555 `INT` (`PERIPH_IO_INT`, input) | **Corrects an earlier version of this doc**, which called this pin spare/freed for a heartbeat LED that never materialized — live schematic trace shows it's actually wired to the I2C GPIO expander's own interrupt output, so firmware can be notified on any expander input change (DIP switches, buttons) instead of polling over I2C. No JTAG-header tradeoff either way — ESP32-S3's native USB has a built-in USB Serial/JTAG peripheral, full OpenOCD-compatible debug access over the same USB-C connector | ✅ |
| 33 | IO40 (MTDO) | WS2812/RGB status LED data (MCU output) | D4, Worldsemi WS2812B-B/W. `DIN` wired to this pin; `DOUT` explicitly no-connect flagged (last/only LED in the chain, safe to leave floating per the part's own datasheet convention); local 0.1µF bypass cap (C52) | ✅ |
| 34 | IO41 (MTDI) | Forward-power comparator output (input, digital) | Fault-flag path: same diode detector as IO4, through a comparator (e.g. LM393-class) against a fixed threshold — reliable "antenna fault, near-zero power leaving" detection independent of ADC noise/averaging | |
| 35 | IO42 (MTMS) | `IO_IN_ON_BATT` (input) | Status tap off the TPS2121 rail muxes — tells firmware whether the board is currently running on battery or USB, e.g. to gate whether a power-off attempt should even try (no point releasing the latch while USB keeps the rails up regardless) | ✅ |
| 36 | RXD0 | USB-UART bridge (CH340C) TX → MCU RX | Own USB connector, separate from native USB — see § MCU in controller_platform.md | ✅ |
| 37 | TXD0 | MCU TX → USB-UART bridge RX | | ✅ |
| 38 | IO2 | MAX17320 ALRT (input, interrupt) | | ✅ |
| 39 | IO1 | PCF8563 INT/alarm (input, interrupt) | Separate from that same INT line's direct hardware path into the power-latch wake-OR (§4.1) — MCU also wants to read it directly once awake | ✅ |
| 40 | GND | Ground plane | | ✅ |
| 41 | EPAD | Ground plane | Thermal/ground pad, solder down | ✅ |

## Open judgment calls (flagging, not blocking)

1. **GPS is UART-only** (no I2C) — simpler, and UART is the standard/expected interface for NMEA/UBX streams. Flag if you wanted I2C instead for some reason.
2. **I2S MCLK not reserved** — not needed: the chosen DAC (PCM5102A) runs its internal PLL off `SCK` tied low, no MCLK required from the MCU (see `audio_ptt_path.md` § 1). IO3, the one pin this would have used if a future DAC swap ever needed it, is now spent on XSMT mute control instead (item 6 below) — a future MCLK requirement would need to reclaim a GPIO from an existing assignment.
3. **Resolved: added a second USB connector for a USB-UART bridge** (CH340C, own port, DTR/RTS auto-reset into EN/IO0), instead of relying solely on native-USB auto-reset-into-bootloader — see § MCU in controller_platform.md for why (native-USB CDC re-enumerates on every reset, and its auto-reset has known version-dependent quirks). BOOT/RESET buttons stay regardless, as the fallback when the bridge isn't plugged in.
4. **Partially resolved: an I2C GPIO expander (PCA9555PWR, `0x20`) is now placed and wired** — but its actual role landed differently than originally planned. The LEDs behind it (4x, on `IO1_0`–`IO1_3`) and buttons (4x onboard + 4x external JST breakouts, on `IO0_x`) are wired as **generic-purpose I/O**, not specifically assigned to "TX-active" and "heartbeat" roles in the schematic — that mapping is now a firmware/config decision, not a hardware one. **The WS2812/RGB status LED (pin 33, D4) is now placed and wired** — it needed a real serial protocol with strict bit timing (RMT peripheral), which a static-register expander can't produce, so it stayed on its own native GPIO as planned.
5. **`IO42` was incorrectly listed as spare** in an earlier pass of this doc — it's actually already wired to `IO_IN_ON_BATT`, added after this table was first written and never back-filled here. Caught by cross-checking the live schematic rather than trusting this doc; worth remembering that this table can drift out of sync with the actual `.kicad_sch` files and should be spot-checked, not assumed current.
6. **`IO39` was also incorrectly listed as spare**, same root cause as item 5 — live schematic trace shows it's wired to the PCA9555's own `INT` output (`PERIPH_IO_INT`), not freed for a heartbeat LED (see the pin table, pin 32). `IO3` (pin 15, the GPIO reference here) is spent on the audio circuit's XSMT mute control — but see item 7 below, a different pin also freed back up since this was written.
7. **`IO15` (pin 8) is spare again** — originally reserved for one leg of a PTT AND-gate (paired with a 74HC123 retrigger pulse on `IO16`). The PTT circuit that actually got built is simpler: a single GPIO (`IO16`) drives a FET (`Q10`) directly, no second GPIO involved. `IO15` was never wired to anything, so it's genuinely available — worth keeping in mind for the still-unbuilt TX-safety timeout (74HC123) stage, though that may end up gating the existing PTT path in series rather than needing a dedicated GPIO of its own.
8. **SA818S `PD`/`H-L` control will reuse PCA9555 expander pins, not native ESP32-S3 GPIOs** — decided this session, not yet built in the schematic. Two of the expander's existing JST-connected channels (currently switch inputs) get reconfigured as outputs: `PD` direct (its existing pull-up conveniently defaults it safe), `H-L` through a small N-FET so the GPIO can only pull it low or let it float — never drive it high, per the SA818 datasheet's explicit "cannot connect to VDD or high CMOS output" constraint. See `sa818_pinout.md`. Doesn't consume a native GPIO, so no change to this table's pin count.
9. **GPS hardware backup-mode power gating (VCC/V_IO cutoff via a FET)** — discussed as the recommended way to force `U18` into its true low-power backup mode (vs. software standby), not yet built. Would use a spare GPIO or the PCA9555 expander to drive the gate — if a native GPIO ends up being used instead of the expander, that would consume `IO15`, the one currently-spare pin. Flagging so that's not forgotten if/when this gets implemented.

**Fresh audit this pass** (ERC-verified, not just net-name lookups — several genuinely-wired local nets don't resolve to a name via simple net queries and were double-checked against ERC's actual unconnected-pin list): confirmed exactly **4 unconnected pins** — `IO15` (spare), `IO46` (strapping, intentionally floating), `IO4` and `IO41` (forward-power ADC/comparator pair, still planning-only). Six other rows (`IO5`, `IO17`, `IO18`, `IO14`, `IO21`, `IO47`) had drifted stale — wired in the schematic but still shown blank/planning here — now corrected to ✅.

35 of 36 available GPIOs now assigned a purpose; **`IO15` (pin 8) is the one genuinely spare pin**. Cross-checking the live schematic (not just this doc) shows all but 4 pins are actually wired today — see the **Wired** column in the table above for the pin-by-pin breakdown; `IO4`/`IO41` (forward-power ADC/comparator) remain the only planning-only reservations among the assigned pins. `PD`/`H-L` on the SA818S itself are separate, non-ESP32-S3 pins (see item 8 above).
