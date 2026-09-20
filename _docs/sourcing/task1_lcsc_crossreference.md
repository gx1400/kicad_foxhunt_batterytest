# BOM / LCSC Cross-Reference — Results

Verification method: see [`_agent_tasks/bom_property_audit.md`](../../_agent_tasks/bom_property_audit.md).
Re-run after any significant schematic change — reference designators on this project have
already churned across three renumbering passes, so treat every designator below as a snapshot
of this date, not a durable identifier.

**Result: 93 of 126 real components have complete `Manf`/`MPN`/`LCSC Part #` data. Zero
Value/MPN/Manufacturer mismatches found against real LCSC data** across every part checked.

## Fixed since the prior pass

- **U13 (CH340C)**: now has Manf=WCH, MPN=CH340C, LCSC=`C84681` — verified correct.
- **R7 (80.6k)**: Manf typo ("YAGE") corrected to YAGEO; MPN=`RT0603BRE0780K6L`, LCSC=`C862291` —
  verified correct.
- **R5, R8 (30.1k)**: both now complete, LCSC=`C137745` — verified correct.
- **R14 (1k)**: LCSC=`C95781` now present.
- **BT2**: now carries the same LCSC ID as BT1 (`C5339083`) instead of being blank — the "copy
  over" is done, though see the standing exception below (that ID is still delisted).

## Open items

- [ ] **The shunt resistor's footprint still doesn't match its sourced part's brand.** Currently
      **R13** (was R11, then R4, now R13 — this is the 0.0025Ω current-sense shunt on the battery
      GND/GNDREF path, identify by function not designator). Footprint
      `R_Shunt_Vishay_WSK2512_6332Metric_T2.21mm` implies Vishay WSK2512 pad geometry; sourced
      part is Milliohm `HoJLR2512-3W-2.5mR-1%` (LCSC `C2904234`). Verify Milliohm's actual pad
      dimensions match before fab.

## Resolved since last pass (2026-09-19)

- **U1, U2, U3 field naming**: normalized from `easyeda2kicad`'s defaults (`Manufacturer`,
  `LCSC Part`) to this project's convention (`Manf`, `LCSC Part #`) — verified via fresh netlist,
  all three consistent (Manf=TI, LCSC Part #=C485916), `Source`/`Imported` traceability fields
  preserved.
- **Stage 1 ILM (R8, 30.1kΩ)**: confirmed intentional — was undersized at the prior 51.1kΩ for
  the combined battery-side current across both downstream rails, per the same reasoning noted
  in the last pass.

## Standing exceptions (see the playbook for the full list/reasoning)

- BT1/BT2: LCSC `C5339083` delisted, known and accepted.
- The two 2W THT bleed resistors (currently R22/R23): intentionally SparkFun-sourced, not
  JLCPCB-verifiable by design.
- Capacitor voltage in `Value` is a minimum, not exact.
- Standard company-name abbreviations (TI/Texas Instruments, etc.) are not mismatches.
- Non-LCSC-sourced parts (F2, J1, J2, etc.) are fine as-is if self-consistent.

## Informational — no action needed

- **F2, J1, J2**: Manf+MPN present and self-consistent (MULTICOMP PRO, Anderson Power Products,
  PHOENIX CONTACT respectively), no LCSC Part# — legitimately non-LCSC-sourced.
- **22 jumpers + 3 test points**: no sourcing fields, expected for generic mechanical parts.
