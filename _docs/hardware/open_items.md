# Open Items / Next Steps

- [ ] Confirm GNDPWR solder-jumper flags are wired to the intended nets (not accidentally re-merging GND/GNDREF) — likely a KiCad global GND power-symbol mixup if it recurs.
- [ ] Populate downstream loads (SA818S, ESP32-S3, GPS) and re-verify rail current budgets against real hardware.
- [ ] First board bring-up: isolate each stage via the power-section jumpers, verify independently, then re-bridge for full-system test.
- [ ] Begin schematic capture for the [controller/peripheral platform](controller_platform.md) — none of it exists in `.kicad_sch` yet, that doc is planning-only.
- [ ] Once the controller section has real current draws, re-verify the existing 5V/2A and 3.3V/1A [regulation](regulation.md) budget still covers it.

See also: [`_docs/sourcing/task1_lcsc_crossreference.md`](../sourcing/task1_lcsc_crossreference.md) for the BOM-verification checklist (blank Value fields, footprint/manufacturer mismatches, missing LCSC data, and the per-sheet capacitor-voltage-is-a-minimum documentation task).
