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

| Pin # | Signal | Connects to | Notes |
|---|---|---|---|
| 1 | GND | Ground plane | |
| 2 | 3V3 | `+3.3V` rail | |
| 3 | EN | Reset circuit: 10kΩ pull-up to 3V3 + 1µF to GND (power-on delay) + manual RESET button to GND + auto-reset transistor from the USB-UART bridge's DTR line (§ MCU in controller_platform.md) | Standard practice |
| 4 | IO4 | Forward-power detector output (ADC, level readback) | **ADC1** channel — deliberately not ADC2; ADC2 is unreliable while WiFi is active. Paired with a comparator fault-flag on IO41 (pin 34) — see §4.8 in README for the dual-path design |
| 5 | IO5 | GPS PPS input | Interrupt-capable, precise edge capture |
| 6 | IO6 | SA818S UART — MCU TX → SA818 RXD | UART2 (software-assigned) |
| 7 | IO7 | SA818S UART — MCU RX ← SA818 TXD | UART2 (software-assigned) |
| 8 | IO15 | SA818S/HT PTT-intent (MCU output) | Feeds one leg of the PTT AND-gate; other leg is the 74HC123 timeout output (§4.10 in README) |
| 9 | IO16 | 74HC123 retrigger/"kick" pulse (MCU output) | Extends the 2-min hardware TX-timeout while a legitimate TX continues |
| 10 | IO17 (U1TXD) | GPS module RXD | UART1, matches datasheet's own U1TXD label |
| 11 | IO18 (U1RXD) | GPS module TXD | UART1 |
| 12 | IO8 | I2C SDA | Shared bus: PCF8563 RTC, MAX17320 fuel gauge, AT24C32D EEPROM |
| 13 | IO19 | **USB_D-** | Native USB, fixed pin |
| 14 | IO20 | **USB_D+** | Native USB, fixed pin |
| 15 | IO3 | TX-active LED (MCU output) | Strapping pin (JTAG source, default floating) — floating default is undisturbed by a normal LED-drive load, safe to reuse |
| 16 | IO46 | **Reserved / NC** | Strapping pin (boot mode + ROM log), leave floating for default boot behavior |
| 17 | IO9 | I2C SCL | Shared bus, see pin 12 |
| 18 | IO10 (FSPICS0) | SD card CS | |
| 19 | IO11 (FSPID) | SD card MOSI (DI) | |
| 20 | IO12 (FSPICLK) | SD card SCLK | |
| 21 | IO13 (FSPIQ) | SD card MISO (DO) | |
| 22 | IO14 | I2S BCLK → audio DAC | DAC part TBD (§4.5 in README) |
| 23 | IO21 | I2S LRCLK/WS → audio DAC | |
| 24 | IO47 | I2S DOUT → audio DAC | MCLK not reserved yet — add only if the chosen DAC needs it |
| 25 | IO48 | Power-latch hold (MCU output, open-drain) | Diode-ORed with button + PCF8563 INT at the soft-latch enable node (§4.1) |
| 26 | IO45 | **Reserved / NC** | Strapping pin (VDD_SPI voltage), leave floating for default (3.3V) behavior |
| 27 | IO0 | BOOT button (pull to GND) + auto-reset transistor from the USB-UART bridge's RTS line | Strapping pin (boot mode). Manual BOOT+RESET buttons stay as the fallback when the bridge isn't plugged in; the bridge's RTS/DTR pair drives the standard two-transistor auto-reset circuit for one-command `esptool`/PlatformIO uploads |
| 28 | IO35 | — | Not available (internal PSRAM) |
| 29 | IO36 | — | Not available (internal PSRAM) |
| 30 | IO37 | — | Not available (internal PSRAM) |
| 31 | IO38 | Push button (input, wake-capable) | General board button — distinct from the BOOT button on IO0 |
| 32 | IO39 (MTCK) | Heartbeat LED (MCU output) | No JTAG-header tradeoff here — ESP32-S3's native USB has a built-in USB Serial/JTAG peripheral, full OpenOCD-compatible debug access over the same USB-C connector, no separate header needed |
| 33 | IO40 (MTDO) | WS2812/RGB status LED data (MCU output) | Same as above |
| 34 | IO41 (MTDI) | Forward-power comparator output (input, digital) | Fault-flag path: same diode detector as IO4, through a comparator (e.g. LM393-class) against a fixed threshold — reliable "antenna fault, near-zero power leaving" detection independent of ADC noise/averaging |
| 35 | IO42 (MTMS) | *Spare* | Free headroom |
| 36 | RXD0 | USB-UART bridge (CP2102N-class) TX → MCU RX | Own USB connector, separate from native USB — see § MCU in controller_platform.md |
| 37 | TXD0 | MCU TX → USB-UART bridge RX | |
| 38 | IO2 | MAX17320 ALRT (input, interrupt) | |
| 39 | IO1 | PCF8563 INT/alarm (input, interrupt) | Separate from that same INT line's direct hardware path into the power-latch wake-OR (§4.1) — MCU also wants to read it directly once awake |
| 40 | GND | Ground plane | |
| 41 | EPAD | Ground plane | Thermal/ground pad, solder down |

## Open judgment calls (flagging, not blocking)

1. **GPS is UART-only** (no I2C) — simpler, and UART is the standard/expected interface for NMEA/UBX streams. Flag if you wanted I2C instead for some reason.
2. **I2S MCLK not reserved** — deferred until the actual DAC part is chosen; some I2S DACs don't need it, some do. The one remaining spare pin (IO42, #35) is available for it if needed later.
3. **Resolved: added a second USB connector for a USB-UART bridge** (CP2102N-class, own port, DTR/RTS auto-reset into EN/IO0), instead of relying solely on native-USB auto-reset-into-bootloader — see § MCU in controller_platform.md for why (native-USB CDC re-enumerates on every reset, and its auto-reset has known version-dependent quirks). BOOT/RESET buttons stay regardless, as the fallback when the bridge isn't plugged in.

35 of 36 available GPIOs assigned; **IO42 (pin 35) is the only pin still genuinely spare** — headroom for whatever comes up during layout/bring-up.
