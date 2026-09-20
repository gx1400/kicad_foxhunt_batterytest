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
- [Controller & peripheral platform](hardware/controller_platform.md) — VRAW soft-latch and PCF8563 RTC implemented; GPS, MCU, audio/PTT, storage, USB power, RF sensing, status LEDs, TX safety, frequency/tone plan, LoRa (skipped), enclosure still planned, not yet in the schematic
- [ESP32-S3 pinout & interconnect plan](hardware/esp32s3_pinout.md)
- [I2C bus — devices & addresses](hardware/i2c_bus.md)
- [Open items / next steps](hardware/open_items.md) — living checklist

## Sourcing (`sourcing/`)

BOM verification and part-sourcing records — datasheets, 3D models, LCSC cross-reference.

- [Task 1 — LCSC/MPN/manufacturer cross-reference](sourcing/task1_lcsc_crossreference.md) — action-item checklist
- [kicad_gx_library datasheets](sourcing/kicad_gx_library_datasheets.md)
- [project_library datasheets](sourcing/project_library_datasheets.md)
- [3D models — search & download status](sourcing/3d_models.md)

## Firmware (`firmware/`)

Reserved for once the PlatformIO project (`src/`) exists — see [firmware/README.md](firmware/README.md).
