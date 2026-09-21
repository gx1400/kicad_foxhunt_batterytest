# Agent Task: Datasheet Localization

Recurring task — re-run this whenever asked to "localize datasheets" or similar, and
periodically as new parts get added to the schematic. Each run should treat prior results
as stale rather than trusted; re-derive everything against the current schematic state.

**This task is authorized to edit each symbol's `Datasheet` property directly — that's its
whole job.** Do not touch any other field, wiring, placement, or library graphics while
running it. This is a narrower, standing exception to this project's normal
"schematic edits are the user's job" rule, scoped specifically to the `Datasheet` field.

## Goal

For every unique part (by MPN, not by reference designator — a part placed 3x should only
be downloaded/processed once) across the schematic whose `Datasheet` field is currently a
**live remote URL** (starts `http://` or `https://`) rather than an already-localized
`${KIPRJMOD}`-relative path:

1. Download the datasheet PDF into this project.
2. If it isn't in English, search for an English-language datasheet for the *same part*
   and use that instead if a genuine one is found.
3. Point every instance of that part's `Datasheet` field at the local, project-relative
   path instead of the remote URL.

## Method

Use the same per-file netlist approach as `_agent_tasks/bom_property_audit.md` (see that
file for the exact tool call and why `get_schematic_component`/`list_schematic_components`
don't work here — they don't expose `Datasheet`/`MPN`/`Manf`/`LCSC Part #`). Cover every
`.kicad_sch` file in `pcb/` — check for new sheets added since this list was last touched:
`foxhunt1.kicad_sch`, `power.kicad_sch`, `battery_18650_input.kicad_sch`,
`battery_powerpole_input.kicad_sch`, `power_regulation.kicad_sch`, `mcu-esp32.kicad_sch`,
`usbc-programming-uart.kicad_sch`, `peripherals.kicad_sch`.

For each unique part found with a remote `Datasheet` URL:

1. **Download.** Fetch the URL. Confirm the response is actually a PDF (not an HTML error
   page or a login/JS-gated wall) before treating it as valid — a mismatched content type
   is a fetch failure, not a datasheet.
2. **Language check.** Read the first 1-2 pages of the downloaded PDF and confirm it's in
   English. If it is, skip to step 4.
3. **Non-English fallback.** If the datasheet isn't in English, web-search for an
   English-language datasheet for the *exact same part* (same MPN, verify it's not a
   similar-but-different part before accepting it). If a genuine English version exists,
   download and use that instead of the original. If none can be found after a reasonable
   search effort, fall back to the non-English PDF — but flag it clearly in the tracking
   doc (see Reporting) so a human knows to revisit it, rather than silently accepting it.
4. **Save location** — mirrors this project's existing two-library split
   (see `CLAUDE.md`):
   - Symbols whose `lib_id` belongs to `kicad_gx_library:` → save under
     `pcb/libs/kicad_gx_library/datasheets/<Manufacturer>/<MPN>.pdf`, tracked in
     `_docs/sourcing/kicad_gx_library_datasheets.md`.
   - Everything else → save under `pcb/project_library/datasheets/<Manufacturer>/<MPN>.pdf`,
     tracked in `_docs/sourcing/project_library_datasheets.md`.
   - `<Manufacturer>` is the part's own `Manf` field text with spaces replaced by
     underscores (mirror existing folder names exactly — don't invent a different
     casing/normalization scheme than what's already on disk).
   - `<MPN>.pdf` — sanitize characters that aren't valid in a filename (e.g. `/` in
     `WS2812B-B/W` → `WS2812B-B_W`), but keep it recognizably the MPN.
5. **Update the schematic.** Set `Datasheet` to `${KIPRJMOD}/project_library/datasheets/...`
   (or the `kicad_gx_library` equivalent) on **every instance of that part across every
   file it appears in** — not just the first one found. Use
   `batch_edit_schematic_components` per file for this.
6. **Update the tracking doc.** Append/update a row in the relevant doc
   (`project_library_datasheets.md` or `kicad_gx_library_datasheets.md`), matching their
   existing table format (Symbol/lib_id, Refs, MPN, Manf, LCSC #, Source URL, Local path,
   Linked). Note the language fallback explicitly if step 3 had to substitute a different
   source than the original URL, or if step 3 failed and a non-English datasheet was kept.

## Standing exceptions — don't re-attempt every run

- **BT1/BT2 (BH-18650-PC battery holder)** and **J3 (1377G12-BK connector)** — no
  downloadable datasheet source found in prior passes (`project_library_datasheets.md`
  § Gaps). Don't burn search effort re-attempting these every run; a quick retry is fine
  if you're already touching this doc for other reasons, but don't treat it as required.
- **R16, R7** — linked to a generic Vishay DCRCWE3-series datasheet, not a part-specific
  one (no confirmed MPN ever recorded for these two). Leave as-is; this is a sourcing gap
  from the BOM audit task, not a datasheet-localization problem to solve here.
- **Non-LCSC-sourced parts already on a manufacturer's own site** (vs. a dead-end host) —
  still worth localizing if the URL is live and points to a real PDF; the "non-LCSC" flag
  in the BOM audit is about sourcing, not about whether the datasheet itself should be
  localized.
- **VHF RF transceiver module (SA818S-V, DNP)** — sourced outside LCSC/JLCPCB entirely
  per `ic_inventory.md`; skip unless a real datasheet URL is already present to localize.

## Reporting

Structured, not narrative:

1. Overall stats — total unique parts checked, how many already had a localized
   `${KIPRJMOD}`-relative path (skipped, nothing to do), how many were newly localized
   this run.
2. Per part newly localized: MPN, Manf, which library folder it landed in, whether the
   original datasheet was already English or a substitute had to be found.
3. Failures — a remote URL that couldn't be fetched at all (dead link, JS-gated, blocked),
   or a non-English datasheet where no English substitute could be found (flag these,
   don't silently keep going).
4. Anything else notable (e.g. a datasheet that turned out to be for a different part than
   the MPN claims — that's a BOM-audit-task finding, not this task's job to fix, but worth
   surfacing since you'll be looking right at it).

## Scale note

Same shape as the BOM property audit — potentially 20-40+ unique parts across 7-8 files,
one download (and possibly a language-fallback web search) per part. Treat as a
background/forked task rather than doing it inline, to keep the bulk PDF-fetch tool output
out of the main conversation's context.
