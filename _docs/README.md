# Docs Index

Everything below the top-level [README](../README.md). Grouped by topic — add new docs to the
matching folder and list them here.

## Hardware (`hardware/`)

Design decisions and reference detail for the PCB. Start with the architecture overview, then
drill into whichever subsystem you need.

- [Power architecture overview](hardware/power_architecture.md) — the two-source-combine-then-regulate diagram, links to the three sections below
- [Battery protection & fuel gauge (MAX17320)](hardware/battery_protection.md)
- [Dual-input power combining](hardware/power_input_combining.md)
- [Regulation — VRAW → 5V/3.3V rails, downstream loads](hardware/regulation.md)
- [Controller & peripheral platform](hardware/controller_platform.md) — VRAW soft-latch, PCF8563 RTC, PCA9555 I2C GPIO expander, WS2812 status LED, CAT24C32YI-GT3 EEPROM, SD card (SPI), MCU (ESP32-S3 + dual-USB), USB power path, GPS, and audio/PTT DAC path implemented; SA818S itself, the external-HT jack, RF sensing, TX safety, frequency/tone plan, LoRa (skipped), enclosure still planned
- [ESP32-S3 pinout & interconnect plan](hardware/esp32s3_pinout.md)
- [Audio / PTT path — I2S DAC, ESP32-S3, SA818S, external HT](hardware/audio_ptt_path.md) — PCM5102A DAC + attenuator + switched external-HT jack (J11) + PTT FET (Q10) implemented
- [SA818S-V pinout & interconnect plan](hardware/sa818_pinout.md) — module placed with UART/PTT/MIC_IN wired; VBAT, ANT, PD, H/L still open
- [VHF matching network & RF forward-power detector](hardware/vhf_matching_detector.md) — parts, layout, and sourcing for the antenna matching ladder and diode-detector power-sensing tap; not yet in the schematic
- [GPS — u-blox MAX-M10S, ESP32-S3, backup power, SMA antenna](hardware/gps_path.md) — implemented, including the CR2032/supercap backup network and RF bias-tee
- [I2C bus — devices & addresses](hardware/i2c_bus.md)
- [IC inventory & alternatives](hardware/ic_inventory.md) — every IC/module used or planned, LCSC MPN/price/stock, and 1-4 scored alternates per chip
- [Open items / next steps](hardware/open_items.md) — living checklist

## Sourcing (`sourcing/`)

BOM verification and part-sourcing records — datasheets, 3D models, LCSC cross-reference.

- [Task 1 — LCSC/MPN/manufacturer cross-reference](sourcing/task1_lcsc_crossreference.md) — action-item checklist
- [kicad_gx_library datasheets](sourcing/kicad_gx_library_datasheets.md)
- [project_library datasheets](sourcing/project_library_datasheets.md)
- [3D models — search & download status](sourcing/3d_models.md)

## Firmware (`firmware/`)

Reserved for once the PlatformIO project (`src/`) exists — see [firmware/README.md](firmware/README.md).
