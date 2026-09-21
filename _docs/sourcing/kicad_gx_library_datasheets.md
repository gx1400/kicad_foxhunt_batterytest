# kicad_gx_library — Symbol Datasheets

For each symbol whose definition lives in `pcb/libs/kicad_gx_library/GX.kicad_sym` (not to be
confused with symbols that merely *use a footprint* from that library — see
`project_library_datasheets.md` for those), confirm/fetch an English-language datasheet, store
it under `pcb/libs/kicad_gx_library/datasheets/`, and link it both at the library-symbol-default
level and at each placed schematic instance.

8 symbols now live in this library. The first 4 (Results table below) were done in an earlier
pass; `BAS116LT1G`, `BS-08-B2AA020-R`, `CAT24C32YI-GT3`, and `TPS2121RUXR` were added since, and
localized in the most recent datasheet-localization run (see
`_agent_tasks/datasheet_localization.md`).

## Results

| Symbol | Refs (sheet) | MPN | Manf | LCSC# | Local path | Library-level linked | Instance-level linked |
|---|---|---|---|---|---|---|---|
| `8205A` | Q2 (battery_18650_input) | 8205A | ALJ | C22458966 | `pcb/libs/kicad_gx_library/datasheets/JSMICRO/8205A.pdf` | ✅ | ✅ |
| `LM74610` | U7, U10 (foxhunt1) | LM74610QDGKRQ1 | TI | C2649431 | `pcb/libs/kicad_gx_library/datasheets/Texas_Instruments/LM74610QDGKRQ1.pdf` | ✅ | ✅ (both refs) |
| `MAX17320` | U2 (battery_18650_input) | MAX17320G22+ | MAXIM | C2914309 | `pcb/libs/kicad_gx_library/datasheets/Analog_Devices/max17320.pdf` | ✅ | ✅ |
| `POLY_FUSE` | F2 (battery_18650_input) | CLM1612P1412 | PTTC(Polytronics Tech) | C5353611 | `pcb/libs/kicad_gx_library/datasheets/Polytronics/CLM1612-12A.pdf` | ✅ | ✅ |
| `BAS116LT1G` | D2, D3 (peripherals) | BAS116LT1G | onsemi | C232527 | `pcb/libs/kicad_gx_library/datasheets/onsemi/BAS116LT1G.pdf` | ✅ | ✅ (both refs) |
| `BS-08-B2AA020-R` | BT3 (peripherals) | BS-08-B2AA020-R | MYOUNG(美阳) | C964787 | `pcb/libs/kicad_gx_library/datasheets/MYOUNG/BS-08-B2AA020-R.pdf` | ✅ | ✅ |
| `CAT24C32YI-GT3` | U16 (peripherals) | CAT24C32YI-GT3 | onsemi | C94264 | `pcb/libs/kicad_gx_library/datasheets/onsemi/CAT24C32YI-GT3.pdf` | ✅ | ✅ |
| `TPS2121RUXR` | U1, U2, U3 (power) | TPS2121RUXR | TI | C485916 | `pcb/libs/kicad_gx_library/datasheets/Texas_Instruments/TPS2121RUXR.pdf` | ✅ | ✅ (all 3 refs) |

All `Datasheet` properties (library-default and per-instance) now use
`${KIPRJMOD}/libs/kicad_gx_library/datasheets/...` relative paths — self-contained, matches
this project's established submodule-portability convention, no absolute paths or external
URLs left in the design.

Verified with `run_erc` on the project root after linking: **19 pre-existing errors** (unused
ESP32-S3 GPIOs, unrelated to this task), **0 warnings**.

## What was downloaded vs. already present (most recent pass)

- All 4 new symbols' original schematic `Datasheet` fields were LCSC `product-detail` pages
  (JS-rendered viewer wrappers), not direct files, or (for `BS-08-B2AA020-R`) an
  `lcsc.com/datasheet/*.pdf` redirect page — same pattern as the original 4. Resolved by
  extracting the real `datasheet.lcsc.com/datasheet/pdf/<hash>.pdf` CDN link embedded in each
  page's HTML and downloading that directly.
- `BAS116LT1G`, `CAT24C32YI-GT3`, `TPS2121RUXR` — all confirmed genuinely English on download,
  no substitution needed.
- `BS-08-B2AA020-R` — a mechanical drawing (MYOUNG battery holder), not prose; predominantly
  English (all dimensions, ordering info, tolerances, material notes), with a handful of small
  Chinese annotations (department stamp, a couple of field labels) alongside their English
  equivalents. Accepted as-is — same judgment call as the two bilingual/mixed datasheets noted
  below, not treated as a fallback-required case since the technical content is fully usable in
  English.

## Language caveat (not blocking, worth knowing)

Three of these datasheets are **bilingual/mixed**, not pure English:
- `8205A.pdf` (ALJ) — English title/headline, but the features section and some body text
  is Chinese. ALJ is a Chinese manufacturer; a pure-English alternate wasn't readily found.
- `CLM1612-12A.pdf` (Polytronics) — appears to be primarily a Chinese-market document. An
  attempt to fetch Polytronics' own English literature timed out (network); not replaced.
- `BS-08-B2AA020-R.pdf` (MYOUNG) — see above; mostly-English mechanical drawing with minor
  Chinese annotations.

Neither blocks usability (key tables/pinouts are numeric/English-labeled either way), but
flagging per the task's "prefer English" instruction in case a cleaner English source is
wanted later.

## Note for the coordinator

`pcb/libs/kicad_gx_library` is a git submodule with its own remote (`gx1400/kicad_gx_library`).
The datasheet additions and the `GX.kicad_sym` property edits (both the original 4-symbol pass
and the most recent 4-symbol addition) are uncommitted changes in that submodule's working
tree — per this project's established two-repo pattern (see `CLAUDE.md`), these need their own
commit+push inside the submodule, then a submodule-pointer bump commit in the parent repo. Not
done here per instructions (no commits from this task).
