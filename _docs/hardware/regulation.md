# Regulation — VRAW → 5V and 3.3V Rails

Two **independent** buck (TPS563201) + linear-LDO cascades, rather than one shared intermediate rail, so each LDO stage runs close to its own output voltage and wastes minimal power as heat.

| | 5V rail (→ SA818S) | 3.3V rail (→ ESP32-S3, GPS) |
|---|---|---|
| Buck stage | TPS563201, 6.67V intermediate | TPS563201, 4.78V intermediate |
| Inductor | 3.3µH (Chilisin MHCI06030-3R3M-R8) | 3.3µH (same part) |
| Feedback (R_top / R_bottom) | 76.8kΩ / 10kΩ, both ±0.1% thin film (Yageo RT0603, matched tolerance) | 52.3kΩ / 10kΩ, both ±0.1% thin film (Yageo RT0603, matched tolerance) |
| Output caps | 2× 22µF/16V X7R ceramic | 2× 22µF/16V X7R ceramic |
| LDO | LM1085-5.0 | AMS1117-3.3 |
| LDO output cap | 22µF/16V **aluminum electrolytic** (~200mΩ ESR) | 0.1µF ceramic + 22µF/16V aluminum electrolytic (~200mΩ ESR) |
| Design current | 2A (SA818S TX peak ~750mA + margin) | 1A (real worst-case ~470mA) |
| Peak inductor current | ~2.92A | ~1.82A |
| Inductor RMS current | ~2.07A | ~1.11A |
| Output cap ripple current | ~0.531A | ~0.475A |

**Intermediate voltages were bumped up from the original 6V/4.3V targets** (68.1k/46.4k dividers) after checking LDO dropout margin — LM1085's dropout is 1.5V max (spec'd over the full current range) and AMS1117's is 1.3V max but only *up to 0.8A* (this rail runs 1A, where the datasheet only says dropout "will be higher," no number given). At 6V/4.3V the 5V rail had just 1.0V of raw headroom — under the LM1085's own spec — so both dividers were recalculated for 0.768V·(1+R_top/R_bottom) against actual TPS563201 reference voltage and re-sourced as precision (±0.1%) matched-tolerance pairs rather than the original 1% singles, so the divider itself no longer adds meaningfully to the output-voltage error budget.

**Margin is still tight, worst case:** at the buck's own ±2% feedback tolerance, the 5V rail's worst-case low intermediate (~6.53V) leaves only ~30mV over the LM1085's 1.5V max dropout spec. The 3.3V rail's actual dropout at its full 1A load is not given in the AMS1117 datasheet (only characterized to 0.8A) — current-limit is spec'd as low as 900mA at a 1.5V differential, which suggests dropout at 1A isn't trivially small. **Bench-verify both rails hold regulation at full rated current before trusting these margins in the field.**

**Note on LDO output caps:** both LM1085 and AMS1117 rely on the output cap's ESR for loop stability (opposite requirement from the buck stage's low-ESR-ceramic-friendly D-CAP2 topology) — a low-ESR ceramic reused from the buck section risks oscillation. Aluminum electrolytics in the ~100–300mΩ ESR range satisfy both LDOs' requirements.

**Planned addition:** optional 200mA test-load jumpers with an in-line DMM measurement header on each LDO output, for bench current verification ahead of populating the real downstream loads.
- 5V rail: 24.9Ω, 2W
- 3.3V rail: 16.5Ω, 1W

## Downstream Loads (planned, not yet on this board)

| Device | Rail | Typical | Peak |
|---|---|---|---|
| SA818S (2m RF module) | 5V | ~60mA RX | ~750mA TX |
| ESP32-S3-WROOM-1-N8R8 | 3.3V | ~20–80mA | ~300–400mA (radio TX burst) |
| u-blox MAX-M10S GPS | 3.3V | ~25–45mA | ~50–70mA (acquisition) |

Controller-section peripherals (RTC, GPS, EEPROM, SD card, audio DAC, status LEDs, 74HC123) are all low-current (µA–low-mA class) individually; a full current budget for the controller section is still TBD once that schematic exists and real component picks are finalized. See [Open Items](open_items.md).
