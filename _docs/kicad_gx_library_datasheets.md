# kicad_gx_library — Symbol Datasheets

Task 2 deliverable: for each of the 4 symbols whose definition lives in
`libs/kicad_gx_library/symbols/GX.kicad_sym` (not to be confused with symbols that merely
*use a footprint* from that library — see `project_library_datasheets.md` for those),
confirm/fetch an English-language datasheet, store it under
`libs/kicad_gx_library/datasheets/`, and link it both at the library-symbol-default level
and at each placed schematic instance.

All 5 placements across the schematic already carried real LCSC/MPN/Manf sourcing data
per-instance (established separately in the Task 1 cross-reference) — this task's job was
making sure the actual PDF exists on disk and that both the library defaults and the
instances point at the local copy instead of an external URL.

## Results

| Symbol | Refs (sheet) | MPN | Manf | LCSC# | Local path | Library-level linked | Instance-level linked |
|---|---|---|---|---|---|---|---|
| `8205A` | Q2 (battery_18650_input) | 8205A | ALJ | C22458966 | `libs/kicad_gx_library/datasheets/JSMICRO/8205A.pdf` | ✅ | ✅ |
| `LM74610` | U7, U10 (foxhunt1) | LM74610QDGKRQ1 | TI | C2649431 | `libs/kicad_gx_library/datasheets/Texas_Instruments/LM74610QDGKRQ1.pdf` | ✅ | ✅ (both refs) |
| `MAX17320` | U2 (battery_18650_input) | MAX17320G22+ | MAXIM | C2914309 | `libs/kicad_gx_library/datasheets/Analog_Devices/max17320.pdf` | ✅ | ✅ |
| `POLY_FUSE` | F2 (battery_18650_input) | CLM1612P1412 | PTTC(Polytronics Tech) | C5353611 | `libs/kicad_gx_library/datasheets/Polytronics/CLM1612-12A.pdf` | ✅ | ✅ |

All `Datasheet` properties (library-default and per-instance) now use
`${KIPRJMOD}/libs/kicad_gx_library/datasheets/...` relative paths — self-contained, matches
this project's established submodule-portability convention, no absolute paths or external
URLs left in the design.

Verified with `run_erc` on the project root after linking: **0 errors, 0 warnings**.

## What was downloaded vs. already present

- **LM74610QDGKRQ1.pdf** — newly downloaded this task. The schematic's recorded LCSC URL
  (`lcsc.com/datasheet/C2649431.pdf`) is a JS-rendered viewer page, not a direct file — had
  to find the underlying CDN asset. The first successful fetch turned out to be the
  **Chinese-language** TI document (ZHCSE83A); replaced with the official English TI
  literature (`ti.com/lit/ds/symlink/lm74610-q1.pdf`, SNOSCZ1B) per the "prefer English"
  instruction.
- **8205A.pdf, max17320.pdf, CLM1612-12A.pdf** — already present from earlier project setup, not re-downloaded.

## Language caveat (not blocking, worth knowing)

Two of the pre-existing datasheets are **bilingual/mixed**, not pure English:
- `8205A.pdf` (ALJ) — English title/headline, but the features section and some body text
  is Chinese. ALJ is a Chinese manufacturer; a pure-English alternate wasn't readily found.
- `CLM1612-12A.pdf` (Polytronics) — appears to be primarily a Chinese-market document. An
  attempt to fetch Polytronics' own English literature timed out (network); not replaced.

Neither blocks usability (key tables/pinouts are numeric/English-labeled either way), but
flagging per the task's "prefer English" instruction in case a cleaner English source is
wanted later.

## Note for the coordinator

`libs/kicad_gx_library` is a git submodule with its own remote (`gx1400/kicad_gx_library`).
The datasheet addition and the 4 `GX.kicad_sym` property edits are uncommitted changes in
that submodule's working tree — per this project's established two-repo pattern (see
`CLAUDE.md`), these need their own commit+push inside the submodule, then a submodule-pointer
bump commit in the parent repo. Not done here per instructions (no commits from this task).
