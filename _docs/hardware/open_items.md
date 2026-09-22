# Open Items / Next Steps

- [ ] 
- [ ] Populate downstream loads (SA818S, ESP32-S3, GPS) and re-verify rail current budgets against real hardware.
- [ ] First board bring-up: isolate each stage via the power-section jumpers, verify independently, then re-bridge for full-system test.

- [ ] Once the controller section has real current draws, re-verify the existing 5V/2A and 3.3V/1A [regulation](regulation.md) budget still covers it.

- [ ] Firmware for hold-to-power-off — poll GPIO38, time the hold, release GPIO48 when the threshold is met. Nothing here is schematic work; this is a firmware dependency on the hardware above.
- [ ] Firmware-independent hardware force-off — still open, no hardware built yet. If firmware hangs while GPIO48 is asserted, nothing currently overrides it. Leading option: reuse the *spare half* of the 74HC123 dual monostable already planned for TX-safety timeout (§ TX safety in `controller_platform.md`), triggered directly by SW3 (or its isolated `POWER_PB_SIGNAL` node), RC-timed longer than the software hold threshold — zero added GPIO, and the 74HC123 isn't placed in the schematic yet either way. Next thing being worked on.

See also: [`_docs/sourcing/task1_lcsc_crossreference.md`](../sourcing/task1_lcsc_crossreference.md) for the BOM-verification checklist (blank Value fields, footprint/manufacturer mismatches, missing LCSC data, and the per-sheet capacitor-voltage-is-a-minimum documentation task).
