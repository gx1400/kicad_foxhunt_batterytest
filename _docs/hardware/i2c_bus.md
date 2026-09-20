# I2C Bus — Devices & Addresses

One shared I2C bus (`MCU_I2C_SDA`/`MCU_I2C_SCL`, GPIO8/GPIO9 on the ESP32-S3 — see
`esp32s3_pinout.md`), spanning the `Power`, `Peripherals`, and `mcu-esp32` sheets. Pull-ups
(R32/R33, 10kΩ to +3.3V) live once on the `mcu-esp32` sheet, not duplicated per device.

## Devices and addresses (7-bit)

| Device | Address | Notes |
|---|---|---|
| PCF8563 (RTC) | `0x51` | Fixed, not configurable. From NXP datasheet ("read A3h / write A2h"). |
| MAX17320 (fuel gauge) | `0x36` (main registers), `0x0B` (NVM/shadow) | Fixed. NVM address only used for one-time config-wizard writes, not normal operation. Verified against a working driver's actual address constants, not assumed from the datasheet prose alone. |
| AT24C32D (EEPROM) | `0x50` (default) | Adjustable `0x50`–`0x57` via A0–A2 pins if a conflict ever shows up. Not yet in the schematic (planned — see `open_items.md`). |
| I2C GPIO expander (PCF8574/PCA9555-class) | `0x20`–`0x27` | Drives the TX-active and heartbeat status LEDs — see `controller_platform.md` § Status indication. Not yet in the schematic (planned). Exact part/address not finalized. |

No address collisions among the above. GPS (u-blox MAX-M10S) is UART-only in this design,
not on this bus — see the open judgment call in `esp32s3_pinout.md`.

## Bus speed

400kHz (I2C Fast-mode) — within PCF8563's spec (`fSCL` max 400kHz per its datasheet) and
comfortably within MAX17320/AT24C32D's typical Fast-mode support. Re-check against the
GPIO expander's own max `fSCL` once a specific part is chosen.

## Verification note

Addresses above were pulled from real sources (NXP's PCF8563 datasheet directly, and a
working open-source MAX17320 driver's address constants — not assumed from memory), same
verification standard as parts sourcing elsewhere in this project. The EEPROM and GPIO
expander addresses are standard-convention defaults, not yet confirmed against a specific
chosen part/order code.
