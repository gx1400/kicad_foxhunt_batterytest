# I2C Bus — Devices & Addresses

One shared I2C bus (`MCU_I2C_SDA`/`MCU_I2C_SCL`, GPIO8/GPIO9 on the ESP32-S3 — see
`esp32s3_pinout.md`), spanning the `Power`, `Peripherals`, and `mcu-esp32` sheets. Pull-ups
(R32/R33, 10kΩ to +3.3V) live once on the `mcu-esp32` sheet, not duplicated per device.

## Devices and addresses (7-bit)

| Device | Address | Notes |
|---|---|---|
| PCF8563 (RTC) | `0x51` | Fixed, not configurable. From NXP datasheet ("read A3h / write A2h"). |
| MAX17320 (fuel gauge) | `0x36` (main registers), `0x0B` (NVM/shadow) | Fixed. NVM address only used for one-time config-wizard writes, not normal operation. Verified against a working driver's actual address constants, not assumed from the datasheet prose alone. |
| CAT24C32YI-GT3 (EEPROM) | `0x50` | Placed (U16, `Peripherals` sheet). A0–A2 grounded → base address `0x50`; adjustable `0x50`–`0x57` via those pins if a conflict ever shows up. |
| PCA9555PWR (I2C GPIO expander) | `0x20` | Placed (U15, `Peripherals` sheet). A0–A2 grounded → base address `0x20`. Drives 4 general-purpose LEDs and 8 general-purpose switch inputs (not yet assigned specific roles — see `controller_platform.md` § Status indication); own `INT` output wired to the MCU (IO39). |

No address collisions among the above. GPS (u-blox MAX-M10S) is UART-only in this design,
not on this bus — see the open judgment call in `esp32s3_pinout.md`.

## Bus speed

400kHz (I2C Fast-mode) — within PCF8563's spec (`fSCL` max 400kHz per its datasheet),
comfortably within MAX17320/CAT24C32YI-GT3's typical Fast-mode support, and matches
PCA9555PWR's own rated 400kHz `fSCL` (per its datasheet, confirmed when the part was
sourced).

## Verification note

Addresses above were pulled from real sources (NXP's PCF8563 datasheet directly, a
working open-source MAX17320 driver's address constants, and direct pin-trace of the
placed CAT24C32YI-GT3/PCA9555PWR address-select pins in the live schematic — not assumed
from memory), same verification standard as parts sourcing elsewhere in this project.
