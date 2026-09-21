# Agent Task: Footprint Verification

Recurring task — re-run this whenever asked for a "footprint audit" or similar, and
periodically as new parts get added to the schematic. Unlike the other two recurring
audit tasks, this one is **differential**: it maintains a persistent log
(`_docs/sourcing/footprint_verification_log.md`) across runs, and a repeat run only needs
to fully re-analyze parts that are new or changed since the last pass — see "Repeated
runs" below.

**This task never edits any footprint, symbol, or schematic file.** It is read-only
end to end — its entire output is the log table plus a findings summary for a human to
act on. If you find a real mismatch, write it down clearly; do not attempt to fix it.

## Goal

For every unique placed part (by MPN/footprint, not by reference designator), answer
three questions in order and record the result:

1. **Does the declared package match the real part?** Cross-reference the part's real
   package (from its datasheet and/or LCSC listing) against what the schematic assumes
   (the `Package` property if set, and/or what the `Footprint` property's name implies).
2. **Is the footprint used a known, standard one for that package?** i.e. is it one of
   KiCad's own stock library footprints (`Package_TO_SOT_SMD:*`, `Resistor_SMD:*`,
   `Capacitor_SMD:*`, `Package_SO:*`, etc.) that's a well-established, community-vetted
   match for the declared package — or is it a one-off/custom footprint?
3. **For anything that isn't a standard stock footprint** (see "Focus areas" below) —
   does the footprint's actual pad geometry (pitch, pad size, overall body/courtyard
   dimensions, pad count) match the datasheet's own mechanical/package-outline drawing?
4. **For anything `easyeda2kicad`-migrated specifically** — does the resulting
   `.kicad_mod`'s geometry faithfully preserve the *original* EasyEDA source footprint's
   geometry, independent of whatever the datasheet says? This isolates conversion-tool
   errors from datasheet-mismatch errors — two different failure modes, see "Quantitative
   method" below.

**All of the above must be answered with real numbers, not adjectives.** "Pad envelope
looks plausible" is not an acceptable cross-reference result on its own — report the
actual measured value, the actual datasheet-stated value (with its tolerance), and the
delta or ratio between them. A reviewer should be able to see the two numbers and judge
the match themselves, not just trust a verdict word.

## Focus areas — where the real risk is

Standard KiCad stock library footprints (anything under a well-known stock library like
`Package_TO_SOT_SMD`, `Resistor_SMD`, `Capacitor_SMD`, `Package_SO`, `Package_DFN_QFN`,
etc.) are heavily community-vetted — for these, a package-family match (question 2) is
normally enough; don't burn time re-deriving pad pitch from scratch for a plain SOT-23 or
0603 unless something about the specific part looks unusual.

**Spend the real effort on:**
- Anything with a `kicad_gx_library:` footprint (hand-authored or `easyeda2kicad`-migrated
  — check `Source` property; `"easyeda2kicad"` means auto-migrated, no `Source` property
  usually means hand-authored for this project).
- Anything with a `project_library:` or other non-stock footprint reference.
- Any stock-library footprint that seems like an unusual pick for its declared package
  (flag for a closer look even if you don't have time to fully dimension-check it).

## Method

Use the same per-file netlist approach as `_agent_tasks/bom_property_audit.md` for
enumerating parts and their `Manf`/`MPN`/`LCSC Part #`/`Footprint`/`Package` fields. Cover
the same file list (check for new sheets): `foxhunt1.kicad_sch`, `power.kicad_sch`,
`battery_18650_input.kicad_sch`, `battery_powerpole_input.kicad_sch`,
`power_regulation.kicad_sch`, `mcu-esp32.kicad_sch`, `usbc-programming-uart.kicad_sch`,
`peripherals.kicad_sch`.

For each unique part:

1. **Determine the real package.** Check the datasheet (already localized under
   `pcb/project_library/datasheets/` or `pcb/libs/kicad_gx_library/datasheets/` per the
   datasheet-localization task — read it directly rather than re-fetching) and/or the
   LCSC listing for the part's actual package designation. Compare against the
   schematic's own `Package` property (if set) and what the `Footprint` name implies.
2. **Get the footprint's actual geometry.** `get_footprint_info(footprint_path="<lib
   nickname>:<footprint name>", include_pads=true, include_graphics=true)` — takes the
   exact `Library:Footprint` string straight from the component's `Footprint` property,
   no need to resolve a file path yourself. Gives pad positions/sizes (compute pitch from
   consecutive pad centers) and courtyard/silkscreen graphics.
3. **Cross-reference against known footprints (question 2).** `search_footprints(query=
   "<package name>")` or `list_library_footprints` on relevant stock libraries to see
   what KiCad's own standard footprint would be for this package, and compare.
4. **For focus-area parts (question 3): read the datasheet's mechanical/package-outline
   page** (usually near the end, titled something like "Package Outline Dimensions,"
   "Mechanical Data," or a JEDEC drawing) and compare pitch, body size, pad count, and
   overall footprint dimensions against what step 2 returned. Note the datasheet's stated
   tolerance/range, not just a single number, when judging a match. Write the actual
   numbers down (e.g. "pitch: KiCad 1.90mm vs. datasheet 1.90mm±0.10mm → match"), not a
   prose conclusion alone.
5. **For `easyeda2kicad`-migrated parts specifically (question 4) — cross-reference
   against the original EasyEDA source, not just the datasheet:**
   - Fetch `https://easyeda.com/api/products/<LCSC_ID>/components?version=6.4.19`
     (public, unauthenticated, no key needed — the same endpoint `easyeda2kicad` itself
     uses; confirmed reachable in this environment). The footprint's raw pad geometry is
     under `result.packageDetail.dataStr` (`shape` entries for pads, `head` for canvas
     scale info) — in EasyEDA's own internal units, not mm.
   - **Do not assume a fixed EasyEDA-unit-to-mm scale constant** — that just substitutes
     one unverified number for another. Instead, compute **relative dimensions (ratios)**
     from the raw EasyEDA pad coordinates that need no unit conversion at all: e.g.
     pitch÷pad-width, or (pad1-to-pad2 distance)÷(pad2-to-pad3 distance). Compute the
     identical ratios from the real-mm `.kicad_mod` geometry (step 2). Matching ratios
     mean the conversion preserved proportions correctly; diverging ratios mean the
     conversion tool distorted the geometry, regardless of what the datasheet says.
   - This is a second, independent check from step 4 — a part can pass one and fail the
     other (e.g. conversion preserved proportions perfectly, but the *original* EasyEDA
     footprint itself didn't match the real part; or conversion distorted an
     otherwise-correct source). Report both results distinctly, don't merge them into one
     verdict.
6. **Score confidence** per the rubric below and write the row.

## Confidence rubric (0-100%, single number per row)

Start from the applicable band, adjust for real findings:

- **95-100%** — Standard KiCad stock footprint, package family clearly matches the real
  part, nothing unusual. (Dimension-level check not required to reach this band.)
- **70-94%** — Footprint family is right and plausible, but either (a) it's a stock
  footprint for a part with a slightly unusual variant/pinout not independently checked,
  or (b) it's a focus-area footprint (gx-library/easyeda-migrated/custom) whose measured
  dimensions matched the datasheet within its stated tolerance (and, for
  `easyeda2kicad`-migrated parts, whose relative-dimension ratios also matched the
  original EasyEDA source — both checks, not just one).
- **40-69%** — Focus-area footprint that could not be fully dimension-verified (datasheet
  lacked a clear mechanical drawing, EasyEDA source data was unavailable/malformed, or a
  ratio check was inconclusive) — flag for manual review, not confirmed wrong, just not
  confirmed right either.
- **10-39%** — A real, specific numeric discrepancy was found (measured pitch/pad-size/
  body-size outside the datasheet's stated tolerance, *or* a relative-dimension ratio that
  diverges from the original EasyEDA source beyond plausible rounding) but it might still
  function (e.g. a slightly oversized courtyard) — needs a human look before layout.
- **0-9%** — Pad count or fundamental package family mismatch (e.g. footprint is a 3-pin
  SOT-23 for a part that's actually SOT-223-4, or a 0603 footprint on a part that's
  actually SOIC-8) — flag prominently, this would fail assembly.

Always report the actual numbers next to the percentage (measured value, datasheet value
with tolerance, and/or the ratio comparison), not just a prose verdict — the number alone
doesn't tell a reviewer what to go check, and two numbers side by side let them verify
your conclusion without repeating the work.

## Log format

`_docs/sourcing/footprint_verification_log.md`, one row per unique part:

| Manf | MPN | LCSC Part # | Refs | Package (declared/real) | Footprint used | Source | Cross-reference result | Confidence | Last audit |
|---|---|---|---|---|---|---|---|---|---|

- **Source** — `stock` / `kicad_gx_library (hand-authored)` / `kicad_gx_library
  (easyeda2kicad)` / `project_library` / other.
- **Cross-reference result** — the actual numbers, not just a verdict word: measured
  dimension(s) vs. datasheet-stated value with tolerance, and, for
  `easyeda2kicad`-migrated parts, the relative-dimension ratio comparison against the
  original EasyEDA source (report both checks distinctly if both apply — datasheet-match
  and conversion-fidelity are independent questions, one can pass while the other fails).
- **Last audit** — the date of the run that produced or last confirmed this row.

Append a short **Findings** section above the table on every run: anything scoring below
70%, called out explicitly with the ref designators, so it's visible without reading the
whole table.

## Repeated runs — differential, not full re-analysis

On every run after the first:

1. Read the existing log first.
2. Re-enumerate current parts (step 1 above is cheap — always do this fresh, reference
   designators and part lists both drift in this project).
3. For a part whose `MPN`, `LCSC Part #`, and `Footprint` are **unchanged** from its
   existing log row: just bump `Last audit` to today's date and copy the row forward —
   don't re-open its datasheet or re-run `get_footprint_info` on it.
4. For a part that's **new** (no existing row) or **changed** (any of those three fields
   differ from the log) — run the full method above and write a fresh row.
5. For a logged part that **no longer appears** in any schematic — remove its row (or
   move it to a small "no longer placed" note at the bottom, your judgment on which reads
   better) rather than leaving it silently stale.

## Scale note

First run is the expensive one — every unique part gets the full treatment, similar
shape to the other two recurring audits (~60-70 unique parts, one or two datasheet
mechanical-page reads plus one `get_footprint_info` call for anything in a focus area).
Treat it as a background/forked task. Repeat runs should be much cheaper given the
differential behavior above — still fine to fork, but don't expect the same depth of
tool calls unless a lot changed.
