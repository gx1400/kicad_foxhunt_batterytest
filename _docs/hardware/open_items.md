# Open Items / Next Steps

- [ ] Confirm GNDPWR solder-jumper flags are wired to the intended nets (not accidentally re-merging GND/GNDREF) — likely a KiCad global GND power-symbol mixup if it recurs.
- [ ] Populate downstream loads (SA818S, ESP32-S3, GPS) and re-verify rail current budgets against real hardware.
- [ ] First board bring-up: isolate each stage via the power-section jumpers, verify independently, then re-bridge for full-system test.
- [ ] Continue schematic capture for the [controller/peripheral platform](controller_platform.md) — the VRAW soft-latch and PCF8563 RTC are done; GPS, SD card, EEPROM, audio/PTT path, RF power sensing, status LEDs, and TX-safety timeout are still planning-only.
- [ ] Once the controller section has real current draws, re-verify the existing 5V/2A and 3.3V/1A [regulation](regulation.md) budget still covers it.
- [ ] Design a power-*off* path for the VRAW soft-latch — the button/RTC-interrupt/MCU-latch circuit currently only powers the system *on*; shutdown today only happens if firmware voluntarily releases its own GPIO48/Q6 latch-hold, no hold-to-shutdown UX and no hardware-guaranteed force-off if firmware hangs. Leading option: a second pole on the power button (DPST, not the existing SPST SW3) wired to its own 3.3V-pulled-up MCU GPIO, so firmware can time a long-press and release the latch — cheaper on standby current than tapping the existing VRAW-swing wake node with a divider. Still open: whether a firmware-independent hardware force-off (timer/supervisor overriding Q5's gate directly, bypassing a hung MCU) is worth the added complexity.

See also: [`_docs/sourcing/task1_lcsc_crossreference.md`](../sourcing/task1_lcsc_crossreference.md) for the BOM-verification checklist (blank Value fields, footprint/manufacturer mismatches, missing LCSC data, and the per-sheet capacitor-voltage-is-a-minimum documentation task).
