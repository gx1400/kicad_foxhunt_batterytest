# Agent Task: BOM Property Audit

Recurring task — re-run this whenever asked for a "fresh" BOM/property audit, and periodically
as the design changes (new parts added, reference designators renumbered, etc.). Each run should
treat prior results as stale rather than trusted; re-derive everything against the current
schematic state.

## Goal

For every real component across the schematic: confirm `Manf`, `MPN`, and `LCSC Part #` are set,
and for anything with an `LCSC Part #`, verify the real LCSC/JLCPCB part actually matches what
the schematic's `Value` field (and `MPN`) claims.

## Method

**Do not use `get_schematic_component` or `list_schematic_components`** — neither exposes custom
fields (Manf/MPN/LCSC Part #), only Value/footprint/position.

Instead, for each `.kicad_sch` file in `pcb/` (currently: `foxhunt1.kicad_sch`, `power.kicad_sch`,
`battery_18650_input.kicad_sch`, `battery_powerpole_input.kicad_sch`, `power_regulation.kicad_sch`,
`mcu-esp32.kicad_sch`, `usbc-programming-uart.kicad_sch` — check for new sheets added since):

1. Call `mcp__konnect__generate_netlist(schematic=<path>, output=<scratchpad>/<name>.net, format="kicad")`.
2. Read the resulting `.net` file. Each component appears as a `(comp (ref ...) (value ...)
   (footprint ...) (fields (field (name "Manf") "...") (field (name "MPN") "...")
   (field (name "LCSC Part #") "...") ...))` block — everything needed is right there, no other
   tool call required per component.

## Step 1 — Completeness check

For every real component, note whether `Manf`, `MPN`, and `LCSC Part #` are each present and
non-empty.

**Skip:**
- Power symbols (`#PWR*`, `#FLG*`).
- Pure mechanical items with no sourcing intent (solder jumpers, test points) *unless* they
  already have LCSC/MPN data set, in which case audit them too.

Produce a list of references missing one or more of the three fields, grouped by file.

## Step 2 — Cross-reference

For every component with an `LCSC Part #`, call `mcp__konnect__get_jlcpcb_part(lcsc_id=...)` and
compare the real part's description/specs against the schematic's own `Value` and `MPN` fields.
Flag a genuine mismatch only — e.g. Value says "10k" but the LCSC part is actually 4.7kΩ, or the
`MPN` field names one part but the LCSC ID resolves to something else entirely. Also flag an LCSC
ID that comes back not-found/delisted.

## Standing exceptions — do not flag these

- **Capacitor voltage in `Value` is a documented minimum, not exact.** A part rated higher than
  the labeled value (e.g. Value says "0.1u/25V", sourced part is 50V-rated) is correct.
- **Standard company-name abbreviations are equivalent**: TI/Texas Instruments, muRata/Murata
  Electronics, AOS/Alpha & Omega Semiconductor, MAXIM/Analog Devices-Maxim, and similarly obvious
  abbreviation/full-name pairs. Use judgment; don't nitpick naming style.
- **Non-LCSC-sourced parts are fine as-is** (Digikey/Mouser/Newark/SparkFun, etc.) as long as
  their own `MPN`/`Manf` fields are self-consistent. Note them as non-LCSC-sourced in the
  completeness accounting, don't flag missing `LCSC Part #` as an error for these.
- **BT1: LCSC Part# `C5339083` is delisted/not found in the JLCPCB catalog.** Known and accepted —
  don't re-flag as a "not found" defect until it's deliberately re-sourced.
- **BT2: carries the same LCSC Part# as BT1 (`C5339083`)**, the known-delisted MPD BH-18650-PC
  holder — this is correct/intentional (same physical part), not a completeness gap.
- **The two 2W THT power resistors (bleed/pre-load resistors on the 5V/3.3V LDO outputs —
  R13/R14 as of the 2nd audit pass, R22/R23 as of the 3rd; check current designators each run,
  don't trust either pair blindly) are intentionally sourced via SparkFun's own catalog
  (`PROD_ID`), not LCSC/JLCPCB.** Not JLCPCB-verifiable by design — don't flag missing
  Manf/MPN/LCSC Part# for these. Identify by description/value/position (2W THT, on the
  regulated-output bleed network), not by reference designator alone — this design has been
  through three renumbering passes and counting.

<!--
  Add newly-agreed exceptions above this line as they come up, in the same style as the three
  above (what the exception is, and why it's not a defect) — this is the list a future audit
  pass should treat as settled, not re-litigate every time.
-->

## Reporting

Structured, not narrative:

1. Overall stats — total real components audited, how many have all three fields complete, how
   many are legitimately non-LCSC-sourced.
2. Completeness gaps — references missing Manf/MPN/LCSC Part #, per file, naming which field(s).
3. Genuine mismatches — schematic claim vs. actual LCSC data, specifics only, high confidence.
4. Anything else notable that doesn't fit the above (e.g. a footprint implying one manufacturer's
   pad geometry while the sourced part is from a different manufacturer — a real physical risk,
   not just a naming inconsistency).

## Scale note

This covers ~100+ components across 7 files and one JLCPCB lookup per LCSC-sourced part — treat
as a background/forked task rather than doing it inline, to keep the bulk tool output out of the
main conversation's context.
