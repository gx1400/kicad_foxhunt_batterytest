# Project Library Datasheets

Datasheets for every placed-component part that is **not** one of the 4 symbols in
`pcb/libs/kicad_gx_library/` (those are documented separately in
`_docs/sourcing/kicad_gx_library_datasheets.md`). Files live under `pcb/project_library/datasheets/`,
organized into per-manufacturer subfolders. All schematic `Datasheet` fields below use the
`${KIPRJMOD}`-relative path convention.

23 unique parts covering 27 component instances across all 4 schematic sheets. 21 of 23
have a downloaded local datasheet linked in the schematic; 2 have no available source
(flagged below).

| Symbol (lib_id) | Refs | MPN | Manf | LCSC # | Source URL | Local path | Linked |
|---|---|---|---|---|---|---|---|
| `Device:C_Small` | C8, C11, C35, C7, C1, C34, C33, C2, C15, C32 | GRM188R71H104KA93D | Murata Electronics | C57112 | https://www.lcsc.com/datasheet/lcsc_datasheet_... (via wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/GRM188R71H104KA93D.pdf` | Yes — all 10 refs |
| `Device:C_Small` | C29, C31, C30, C28 | GRM31CZ71C226ME15L | Murata Electronics | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/GRM31CZ71C226ME15L.pdf` | Yes — all 4 refs |
| `PCM_SparkFun-Resistor:R` | R4, R2, R1, R3 | RC0805FR-0710KL | YAGEO | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/YAGEO/RC0805FR-0710KL.pdf` | Yes — all 4 refs |
| `Device:C_Small` | C14, C16, C13 | GRM188R71A474KA61D | Murata Electronics | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/GRM188R71A474KA61D.pdf` | Yes — all 3 refs |
| `Device:C_Small` | C18, C19 | GRM21BR71C105KA01L | Murata Electronics | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/GRM21BR71C105KA01L.pdf` | Yes — both refs |
| `Device:C_Small` | C5, C4, C6, C3 | GRM21BZ71E106KE15L | muRata | C237493 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/GRM21BZ71E106KE15L.pdf` | Yes — all 4 refs |
| `Device:C_Small` | C12 | GCM219R71E474KA55D | Murata Electronics | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/GCM219R71E474KA55D.pdf` | Yes |
| `Device:C_Small` | C37, C36 | XT1C220M0506 | JARSON | C5260805 | LCSC (wmsc.lcsc.com CDN, numeric prefix 2304140030) | `pcb/project_library/datasheets/JARSON/XT1C220M0506.pdf` | Yes — both refs |
| `Device:Thermistor` | TH1 | NCP15XH103F03RC | Murata Electronics | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/NCP15XH103F03RC.pdf` | Yes |
| `PCM_SparkFun-Resistor:R` | R13 | RC0805FR-07820RL | YAGEO | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/YAGEO/RC0805FR-07820RL.pdf` | Yes |
| `PCM_SparkFun-Resistor:R` | R8 | RC0805FR-0710RL | YAGEO | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/YAGEO/RC0805FR-0710RL.pdf` | Yes |
| `PCM_SparkFun-Resistor:R` | R9, R10 | RT1206BRD07150RL | YAGEO | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/YAGEO/RT1206BRD07150RL.pdf` | Yes — both refs |
| `PCM_SparkFun-Resistor:R` | R12 | RC0805FR-071KL | YAGEO | *(none — Task 1 finding: LCSC field empty; confirmed C95781 is the correct matching part via `get_jlcpcb_part`, not written to schematic — out of this task's scope)* | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/YAGEO/RC0805FR-071KL.pdf` | Yes |
| `PCM_SparkFun-Resistor:R` | R11 | HoJLR2512-3W-2.5mR-1% | Milliohm | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Milliohm/HoJLR2512-3W-2.5mR-1_.pdf` | Yes |
| `PCM_SparkFun-Resistor:R` | R5 | AC0603FR-0768K1L | YAGEO | C228015 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/YAGEO/AC0603FR-0768K1L.pdf` | Yes |
| `PCM_SparkFun-Resistor:R` | R6 | RC0805FR-0746K4L | YAGEO | C483162 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/YAGEO/RC0805FR-0746K4L.pdf` | Yes |
| `PCM_SparkFun-Resistor:R` | R16, R7 | *(none — no confirmed specific MPN/LCSC; "PROD_ID" field only: RES-07856)* | Vishay (series-level only) | — | Vishay DCRCWE3 series page (generic, not part-specific) | `pcb/project_library/datasheets/Vishay/dcrcwe3_series_generic.pdf` | Yes, but flagged **unconfirmed/generic** — both refs |
| `Regulator_Switching:TPS563201` | U3, U1 | TPS563201DDCR | TI | C116592 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/TI/TPS563201DDCR.pdf` | Yes — both refs |
| `Regulator_Linear:AMS1117-3.3` | U9 | AMS1117-3.3 | Advanced Monolithic Systems | C6186 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Advanced_Monolithic_Systems/AMS1117-3.3.pdf` | Yes |
| `Regulator_Linear:LM1085-5.0` | U8 | LM1085IS-5.0RG | HANSCHIP semiconductor | C5145336 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/HANSCHIP_semiconductor/LM1085IS-5.0RG.pdf` | Yes |
| `Device:L` | L4, L3 | MHCI06030-3R3M-R8 | Chilisin | C108294 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Chilisin/MHCI06030-3R3M-R8.pdf` | Yes — both refs |
| `Device:D` | D2 | SMAJ18A | Littelfuse | — | LCSC (`lcsc_datasheet_...` → transformed to wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Littelfuse/SMAJ18A.pdf` | Yes |
| `Device:Fuse` | F3 | MCCQ-122 | Multicomp Pro | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/MULTICOMP_PRO/MCCQ-122.pdf` | Yes |
| `Connector:Screw_Terminal_01x02` | J4 | MC 1,5/2-GF-3,81 | Phoenix Contact | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/PHOENIX_CONTACT/MCV_1_5__2-GF-3_81.pdf` | Yes |
| `Device:Q_NMOS_GSD` | Q1, Q8, Q9 | AO3400A | Hottech | — | *(pre-existing file, not re-downloaded this task)* | `pcb/libs/kicad_gx_library/datasheets/Hottech/AO3400.pdf` | Yes — all 3 refs (reused existing library file rather than duplicating) |
| — | BT1, BT2 | BH-18650-PC | *(battery holder, no LCSC catalog match found)* | — | **Not found** — not present in JLCPCB parts DB; no other reliable source located | *(none)* | **No — not linked** |
| — | J3 | 1377G12-BK | *(connector, Mouser-listed)* | — | **Not found** — mouser.com blocks automated PDF fetch (returns `text/html` even with full browser headers); no alternate source located | *(none)* | **No — not linked** |

## Gaps

- **BT1, BT2 (BH-18650-PC battery holder)** — no downloadable datasheet source found. Schematic `Datasheet` field left untouched.
- **J3 (1377G12-BK connector)** — Mouser-hosted PDF, blocked automated download. Schematic `Datasheet` field left untouched.
- **R16, R7** — linked to a generic Vishay DCRCWE3-series datasheet, not a confirmed part-specific document (no real MPN/LCSC# was ever recorded for these two references — see Task 1 findings).
- **R12** — LCSC Part # field is empty in the schematic (Task 1 finding); the correct matching LCSC ID (C95781) was identified via `get_jlcpcb_part` for reference but not written back to the schematic, since correcting sourcing data is outside this task's scope.
