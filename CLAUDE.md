# CLAUDE.md — foxhunt1 project notes

Working notes for Claude sessions on this KiCad 10 project. See `README.md` for the
actual hardware design (power architecture, MAX17320 config, regulation stages).
This file is about how the *project and tooling* are set up, and rules learned the
hard way this session.

## Repo / library setup

- Three non-standard libraries are vendored as git submodules under `libs/`:
  `kicad_gx_library` (gx1400's own), `digikey-kicad-library`, `SparkFun-KiCad-Libraries`.
  Clone with `git clone --recurse-submodules`, or `git submodule update --init --recursive`
  on an existing clone.
- Project-level `sym-lib-table` / `fp-lib-table` reference these via `${KIPRJMOD}` —
  no machine-specific global KiCad library setup is required to open this project.
- Standard KiCad libraries (`Device`, `power`, `Resistor_SMD`, etc.) need nothing —
  they ship with any KiCad 10 install.
- If a footprint/symbol "not found" error references `kicad_gx_library`, check
  whether the *submodule's* filenames drifted from what the schematic expects
  (this happened once — files got renamed upstream without updating references).
  Fix by renaming inside the submodule to match, not by editing every reference.

## Schematic hierarchy — root filename is load-bearing

- This project uses KiCad 10's `top_level_sheets` feature. Current structure:
  `foxhunt1.kicad_sch` (root, titled "Power") → `Battery 18650 Input`,
  `Battery_PowerPole_Input`, `Power Regulation` (all proper sub-sheets, not
  siblings). The battery/12V ORing-FET combining circuit lives directly in the
  root sheet since that's the actual merge point.
- **The root schematic file MUST be named `<project-name>.kicad_sch`** — i.e.
  `foxhunt1.kicad_sch` — even though KiCad 10 itself doesn't require this anymore.
  Konnect's (the MCP plugin) project-ownership resolution is hard-coded to the
  pre-KiCad-10 convention (`crates/konnect-core/src/tools/mod.rs`): it computes
  the expected root as `<project>.kicad_pro` → `<project>.kicad_sch` and refuses
  to run ERC or several other hierarchy-aware tools if that file is missing or
  named anything else, with a confusing "cannot establish unique project
  ownership" error. **Do not rename the root sheet away from `foxhunt1.kicad_sch`**,
  no matter what it's titled inside KiCad. This is a Konnect limitation, not a
  KiCad one — native KiCad ERC works fine either way.
- After any schematic restructuring, run `validate_sheet_pins` (0 issues expected)
  and `run_erc` on the root to confirm the hierarchy is intact.

## Net classes / trace widths

Three power netclasses exist, sized via IPC-2221 (external layer, 1oz copper,
10°C rise):

| Class | Width | Covers |
|---|---|---|
| `Power_Combined_3A` | 1.4mm | Everything upstream of/at the battery+12V merge point: raw inputs, cell interconnect, CHG/DIS FET path, ORing FET hops, `PWR_IN_SELECT`, GND/GNDREF |
| `Power_5V_2A` | 0.8mm | 5V rail: buck VIN/OUT, LDO VIN/OUT |
| `Power_3V3_1A` | 0.3mm | 3.3V rail: same stages |

**Rule: every net that matters gets a real label, especially anything downstream
of an isolation jumper (JP*).** KiCad auto-names an unlabeled net after whichever
pin it's nearest to (`Net-(JP9-B)`, `Net-(Q2-S1)`, etc.) — netclass assignment is
name-pattern matching, so an auto-generated name is fragile: rename a component,
re-route near a jumper, and the pattern silently stops matching and the segment
falls back to `Default` (0.2mm) with no error. The fix already applied once:
give the net a real `net_label` at the jumper/component pin
(`5V_BUCK_VIN`, `3V3_LDO_OUT`, `BATT_PCKP`, `12V_RAW`, etc.), then point the
netclass pattern at that stable name instead.

**After adding/moving any jumper or wiring near the power section**, re-run
`get_netclasses` and check for nets that should be in a power class but are
matching `Default` instead — that's the signature of this exact bug recurring.

Board-wide DRC floors are set: `min_clearance`/`min_trace_width` = 0.15mm,
`min_via_drill` = 0.3mm, `min_via_size` = 0.5mm (matches configured JLCPCB fab
constraints). Don't let `Default` netclass silently allow anything below these.

**⚠️ `net_settings` (netclasses + DRC floors) has been silently wiped once
already** by the KiCad GUI saving a stale in-memory copy of `foxhunt1.kicad_pro`
over Konnect's on-disk edits — the GUI had the project open from before the
netclasses existed, and closing/saving it clobbered them back to just `Default`.
**Whenever creating/editing netclasses or design rules via Konnect: confirm
KiCad has the project fully closed first (`open_project` → `open_board_count: 0`
and no stale IPC session), do the edits, verify with `get_netclasses`, and
commit immediately** — don't leave netclass edits sitting uncommitted while
there's any chance the GUI reopens the project and re-saves over them.

## Design conventions (see README for full rationale)

- **GND vs GNDREF are deliberately separate nets**, bridged only by the current-
  sense resistor R3. Don't "fix" an apparent GND/GNDREF split — it's intentional
  (makes the shunt actually measure current instead of being bypassed).
- **Decoupling caps are placed at the point of use, often past an isolation
  jumper**, not on the literal named rail. An automated "rail X has no
  decoupling" finding is often a false positive from this pattern — trace the
  actual net before adding a redundant cap. Verified true positives: none found
  so far; every IC's required cap turned out to already be present when traced.
- **Extensive solder-jumper isolation** (JP-prefixed) throughout the power
  section lets each stage be bench-tested independently. This is why so many
  nets are jumper-separated instead of directly wired — expected, not a defect.

## Known Konnect/KiCad tooling quirks (this environment)

- **Live IPC to KiCad doesn't support `GetOpenDocuments`** in this KiCad 10.0.6 +
  Konnect 0.12.0 combination. Any tool that needs to check what's open in the
  live GUI (`get_component_list`, `find_component`, `run_erc`/`get_nets_list`
  when the board is open, etc.) will fail with `AS_UNHANDLED`. **Workaround: ask
  the user to close the board/project in the KiCad GUI**, confirm via
  `open_project` (should report `open_board_count: 0`), then the same tool works
  via the closed-file fallback path.
- **`update_pcb_from_schematic` is the one exception — it's live-IPC-only with
  no closed-file fallback.** It fails both when the board is open (IPC conflict)
  and when it's closed (no IPC to talk to at all). There is no way to trigger a
  schematic→PCB net/footprint sync from Konnect in this environment. When net
  labels or footprints change in the schematic, ask the user to run KiCad's own
  **Tools → Update PCB from Schematic (F8)** natively, then re-verify from
  Konnect's side afterward.
- **`reload_server` is needed after directly editing `settings.json`** (e.g.
  `jlcpcb_db_path`) — the running Konnect process doesn't pick up config file
  changes on its own. Pass `allow_same_version=true` for a same-version reload.
  Toolsets reset to startup defaults on reload — reload what you need after.
- **Empty string `""` in `settings.json` is not the same as unset/null** for
  Konnect's config — it deserializes to `Some("")`, not `None`, and can cause
  a tool to try writing to an empty path. Remove the key entirely rather than
  blanking its value.
- **A `run_design_review`/DRC/ERC run against only one sheet in a multi-root
  project will produce false positives** for anything living on a sibling
  top-level sheet (flagged as "extra footprint", schematic-parity mismatches,
  etc.) — the tool can only see one sheet's schematic at a time even though the
  board is shared. Not a real defect; just audit each sheet and use judgment
  about what's cross-sheet noise vs. real.
- Konnect's `get_installation_info`/`server_stats` etc. are useful for
  diagnosing "is this actually broken or is it a stale cache" — reach for those
  before assuming a design problem when a tool's behavior looks wrong.

## Workflow rules for this project specifically

- Before any risky rename/delete in the schematic or project files, check
  `git status` and consider whether the change needs a corresponding update
  elsewhere (e.g. `top_level_sheets` in `.kicad_pro`, `Sheetfile` properties in
  parent sheets, netclass patterns, submodule pointers).
- After a schematic hierarchy change: `get_sheet_hierarchy` → `validate_sheet_pins`
  → `run_erc`, in that order, before considering it done.
- Only commit/push when explicitly asked. This project has a submodule
  (`kicad_gx_library`) with its own remote — a fix inside it needs its own
  commit+push *and* a submodule-pointer bump+commit in the parent repo.
