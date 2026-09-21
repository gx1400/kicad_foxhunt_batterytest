# Project Library Datasheets

Datasheets for every placed-component part that is **not** one of the symbols in
`pcb/libs/kicad_gx_library/` (those are documented separately in
`_docs/sourcing/kicad_gx_library_datasheets.md`). Files live under `pcb/project_library/datasheets/`,
organized into per-manufacturer subfolders. All schematic `Datasheet` fields below use the
`${KIPRJMOD}`-relative path convention.

**Scope update:** the original pass covered 23 unique parts / 27 instances across 4 schematic
sheets. A subsequent datasheet-localization run (see `_agent_tasks/datasheet_localization.md`)
covered the full 8-sheet hierarchy and localized every remaining part whose `Datasheet` field
was still a live remote URL — 41 unique parts across `power.kicad_sch`,
`battery_18650_input.kicad_sch`, `power_regulation.kicad_sch`, `mcu-esp32.kicad_sch`,
`usbc-programming-uart.kicad_sch`, and `peripherals.kicad_sch`. Two parts could not be resolved
(see Gaps). The table below now mixes both passes; rows added/updated in the most recent run are
marked accordingly.

| Symbol (lib_id) | Refs | MPN | Manf | LCSC # | Source URL | Local path | Linked |
|---|---|---|---|---|---|---|---|
| `Device:C_Small` | C8, C11, C35, C7, C1, C34, C33, C2, C15, C32 | GRM188R71H104KA93D | Murata Electronics | C57112 | https://www.lcsc.com/datasheet/lcsc_datasheet_... (via wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Murata_Electronics/GRM188R71H104KA93D.pdf` | Yes — all 10 refs *(ref list predates later renumbering passes — identify by MPN, not designator, per this doc's own repeated caveat)* |
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
| `PCM_SparkFun-Resistor:R` | R16, R7 | *(none — no confirmed specific MPN/LCSC; "PROD_ID" field only: RES-07856)* | Vishay (series-level only) | — | Vishay DCRCWE3 series page (generic, not part-specific) | `pcb/project_library/datasheets/Vishay/dcrcwe3_series_generic.pdf` | Yes, but flagged **unconfirmed/generic** — both refs *(note: current live schematic has R7 re-sourced to a real YAGEO part, RT0603BRE0780K6L — see new row below; this old row's "R7" reference is stale, kept for R16 only going forward)* |
| `Regulator_Switching:TPS563201` | U3, U1 | TPS563201DDCR | TI | C116592 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/TI/TPS563201DDCR.pdf` | Yes — both refs *(ref list is stale — current U1/U3 are TPS2121RUXR, see `kicad_gx_library_datasheets.md`; identify this row by MPN)* |
| `Device:L` | L4, L3 | MHCI06030-3R3M-R8 | Chilisin | C108294 | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Chilisin/MHCI06030-3R3M-R8.pdf` | Yes — both refs |
| `Device:D` | D2 | SMAJ18A | Littelfuse | — | LCSC (`lcsc_datasheet_...` → transformed to wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/Littelfuse/SMAJ18A.pdf` | Yes *(ref is stale — current D2 is BAS116LT1G, see `kicad_gx_library_datasheets.md`; identify by MPN)* |
| `Device:Fuse` | F3 | MCCQ-122 | Multicomp Pro | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/MULTICOMP_PRO/MCCQ-122.pdf` | Yes |
| `Connector:Screw_Terminal_01x02` | J4 | MC 1,5/2-GF-3,81 | Phoenix Contact | — | LCSC (wmsc.lcsc.com CDN) | `pcb/project_library/datasheets/PHOENIX_CONTACT/MCV_1_5__2-GF-3_81.pdf` | Yes *(ref is stale — current J4 is a JST 3-pin connector, see new row below; identify by MPN)* |
| `Device:Q_NMOS_GSD` | Q1, Q8, Q9 | AO3400A | Hottech | — | *(pre-existing file, not re-downloaded this task)* | `pcb/libs/kicad_gx_library/datasheets/Hottech/AO3400.pdf` | Yes — all 3 refs (reused existing library file rather than duplicating) |
| `Regulator_Linear:LM1085-5.0` | U8 | LM1085IS-5.0RG | TI | C2865978 | *(datasheet-localization pass)* — original schematic URL was TI's **Chinese-region mirror** (`ti.com/cn/.../lm1084.pdf`) pointing at the wrong family sibling (LM1084, 5A, not the actual LM1085, 3A) | `pcb/project_library/datasheets/TI/LM1085IS-5.0RG.pdf` (genuine English TI LM1085 datasheet, `ti.com/lit/ds/symlink/lm1085.pdf`) | **Yes — updated this pass.** Supersedes this doc's earlier claim of a `HANSCHIP_semiconductor` local path, which no longer matched the live schematic (drifted back to a remote, wrong-language, wrong-sibling-part URL at some point after the original pass — worth a BOM-audit look at how, not fixed here beyond re-localizing correctly) |
| — | BT1, BT2 | BH-18650-PC | MPD (Memory Protection Devices) | C5339083 (delisted, known/accepted per `bom_property_audit.md`) | *(datasheet-localization pass — found via LCSC's redirect page, not the direct `.pdf` link)* | `pcb/project_library/datasheets/MPD/BH-18650-PC.pdf` | **Yes — resolved this pass.** Previously flagged not-found; a real English PDF exists on LCSC's CDN even though the LCSC catalog listing itself is delisted |
| — | J3 | 1377G12-BK | Anderson Power Products, Inc. | — | **Not found** — mouser.com blocks automated PDF fetch (returns `text/html` even with full browser headers); re-attempted this pass, still blocked | *(none)* | **No — not linked** |
| `Device:R_US` | R2, R3 | 0603WAF2372T5E | UNI-ROYAL | C22912 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/UNI-ROYAL/0603WAF2372T5E.pdf` | Yes — both refs |
| `Device:Polyfuse` | F3 | 0805L050/16XR | LUTE | C22435898 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/LUTE/0805L050_16XR.pdf` | Yes |
| `Transistor_FET:AO3401A` | Q5, Q8 | AO3407A | AOS | C15155 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/AOS/AO3407A.pdf` | Yes — both refs |
| `Device:LED` | D7 | APT2012LZGCK | Kingbright | C5569446 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Kingbright/APT2012LZGCK.pdf` | Yes |
| `Device:LED` | D9 | APTD2012LQBC/D | Kingbright | C5879058 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Kingbright/APTD2012LQBC_D.pdf` | Yes |
| `Device:LED` | D8 | APTD2012LSURCK | Kingbright | C5366382 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Kingbright/APTD2012LSURCK.pdf` | Yes |
| `Device:LED` | D10 | APTD2012LSYCK | Kingbright | C5588998 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Kingbright/APTD2012LSYCK.pdf` | Yes |
| `Connector_Generic:Conn_01x03` | J4, J5, J6, J7 | B3B-PH-SM4-TB(LF)(SN) | JST | C160353 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/JST/B3B-PH-SM4-TB_LF__SN_.pdf` | Yes — all 4 refs |
| `Transistor_FET:BSS138` | Q7, Q6 | BSS138LT1G | onsemi | C82045 | *(datasheet-localization pass — original onsemi.com URL returned "Access Denied"; recovered via LCSC's own hosted mirror)* | `pcb/project_library/datasheets/onsemi/BSS138LT1G.pdf` | Yes — both refs (Q7 peripherals, Q6 mcu-esp32) |
| `Interface_USB:CH334R` | U11 | CH334R | WCH | C4154405 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/WCH/CH334R.pdf` | Yes |
| `Interface_USB:CH340C` | U13 | CH340C | WCH | C84681 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/WCH/CH340C.pdf` | Yes |
| `Switch:SW_DIP_x04` | SW4 | DSHP04TS-S | XKB Connection | C319050 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/XKB_Connection/DSHP04TS-S.pdf` | Yes |
| `PCM_Espressif:ESP32-S3-WROOM-1` | U10 | ESP32-S3-WROOM-1-N8R8 | ESPRESSIF | C2913201 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/ESPRESSIF/ESP32-S3-WROOM-1-N8R8.pdf` | Yes |
| `Device:C_Small` | C1, C2, C3 | GRM1885C1H103JA01D | muRata | C85973 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Murata_Electronics/GRM1885C1H103JA01D.pdf` | Yes — all 3 refs *(distinct part from the C57112/GRM188R71H104KA93D row above despite similar-looking MPN — 10nF vs 100nF)* |
| `Device:C_Small` | C32, C36 | GRM21BR71H105KA12L | muRata | C77083 | *(datasheet-localization pass — also fixed C36's Datasheet, which pointed to a different capacitor's PDF entirely; a BOM-audit-adjacent finding, corrected as part of this pass)* | `pcb/project_library/datasheets/Murata_Electronics/GRM21BR71H105KA12L.pdf` | Yes — both refs |
| `Switch:SW_Push` | SW3, SW7, SW8, SW9, SW10, SW1, SW2 | K2-1102SP-A4SC-04 | Korean Hroparts Elec | C83916 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Korean_Hroparts_Elec/K2-1102SP-A4SC-04.pdf` | Yes — all 7 refs (power, peripherals ×4, usbc-programming-uart ×2) |
| `PCM_SparkFun-Regulator:LM1117-3.3` | U9, U12 | LM1117IMPX-3.3/NOPB | TI | C23984 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/TI/LM1117IMPX-3.3_NOPB.pdf` | Yes — both refs. **Supersedes** the old `Regulator_Linear:AMS1117-3.3` row from the original pass — U9 (power_regulation) and U12 (usbc-programming-uart) were swapped from AMS1117-3.3 to this part earlier in this project's history |
| `Transistor_BJT:MMBT2222A` | Q3, Q4 | MMBT2222ALT1G | onsemi | C82460 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/onsemi/MMBT2222ALT1G.pdf` | Yes — both refs |
| `PCM_SparkFun-Clock:Crystal` | Y2 | NX3215SA-32.768K-STD-MUA-9 | NDK | C519280 | *(datasheet-localization pass — LCSC-hosted copy was Japanese; replaced with NDK's own English catalog page, `ndk.com/images/products/catalog/c_NX3215SA_e.pdf`, which covers this exact spec variant)* | `pcb/project_library/datasheets/NDK/NX3215SA-32.768K-STD-MUA-9.pdf` | Yes |
| `Interface_Expansion:PCA9555PW` | U15 | PCA9555PWR | TI | C2864778 | *(datasheet-localization pass — LCSC-hosted copy was a Chinese-translated TI document; replaced with TI's genuine English original, SCPS131, `ti.com/lit/pdf/scps131`)* | `pcb/project_library/datasheets/TI/PCA9555PWR.pdf` | Yes |
| `Timer_RTC:PCF8563T` | U14 | PCF8563T/5,518 | NXP | C7440 | *(datasheet-localization pass — nxp.com blocked automated fetch entirely ("Page not available"); LCSC's mirror for this part resolved to a different manufacturer's clone datasheet (Tudi Semiconductor "TUDI-PCF8563", not genuine NXP) and was rejected; recovered via RS Components' mirror of the real NXP document, Rev. 05)* | `pcb/project_library/datasheets/NXP/PCF8563T_5_518.pdf` | Yes |
| `Device:R_US` | R37 | RC0603DR-07590RL | YAGEO | C859077 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603DR-07590RL.pdf` | Yes |
| `Device:R_US` | R31 | RC0603FR-07100KL | YAGEO | C14675 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603FR-07100KL.pdf` | Yes |
| `Device:R_US` | R39, R40, R41, R42 | RC0603FR-071KL | YAGEO | C22548 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603FR-071KL.pdf` | Yes — all 4 refs |
| `Device:R_US` | R29 | RC0603FR-071ML | YAGEO | C105578 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603FR-071ML.pdf` | Yes |
| `Device:R_US` | R5, R8 | RC0603FR-0730K1L | YAGEO | C137745 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603FR-0730K1L.pdf` | Yes — both refs |
| `Device:R_US` | R36, R38 | RC0603FR-07330RL | YAGEO | C105881 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603FR-07330RL.pdf` | Yes — both refs |
| `Device:R_US` | R35 | RC0603FR-07750RL | YAGEO | C114635 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603FR-07750RL.pdf` | Yes |
| `Device:R_US` | R24, R25 | RC0603JR-075K1L | YAGEO | C14677 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RC0603JR-075K1L.pdf` | Yes — both refs |
| `Device:R_US` | R26, R27, R28 | RC0805FR-0710KL | YAGEO | C84376 | *(datasheet-localization pass — distinct LCSC ID from the earlier RC0805FR-0710KL/C57112-adjacent row above; same MPN text, different listing)* | `pcb/project_library/datasheets/YAGEO/RC0805FR-0710KL.pdf` | Yes — all 3 refs |
| `Device:R_US` | R30, R34 | RT0603BRD0710KL | YAGEO | C95204 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RT0603BRD0710KL.pdf` | Yes — both refs |
| `Device:R_US` | R1, R4 | RT0603BRE075KL | YAGEO | C862232 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RT0603BRE075KL.pdf` | Yes — both refs |
| `Device:R_US` | R7 | RT0603BRE0780K6L | YAGEO | C862291 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/YAGEO/RT0603BRE0780K6L.pdf` | Yes |
| `74xGxx:74LVC1G08` | U4 | SN74LVC1G08DBVR | TI | C7666 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/TI/SN74LVC1G08DBVR.pdf` | Yes |
| `PCM_SparkFun-Clock:Crystal` | Y1 | TXM12M0004252DBCEO00T | Yajingxin | C284154 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Yajingxin/TXM12M0004252DBCEO00T.pdf` | Yes |
| `LED:WS2812B` | D4 | WS2812B-B/W | Worldsemi | C114586 | *(datasheet-localization pass)* | `pcb/project_library/datasheets/Worldsemi/WS2812B-B_W.pdf` | Yes |

## Gaps

- **J3 (1377G12-BK connector)** — Mouser-hosted PDF, blocked automated download. Re-attempted
  during the datasheet-localization pass, still blocked. Schematic `Datasheet` field left
  untouched (still the Mouser URL).
- **R16, R7** — R16 is still linked to a generic Vishay DCRCWE3-series datasheet, not a
  confirmed part-specific document (no real MPN/LCSC# was ever recorded for it — see Task 1
  findings). **R7 has since been re-sourced** to a real YAGEO part (RT0603BRE0780K6L, C862291)
  with a genuine part-specific English datasheet, now localized — see the row above; this old
  gap no longer applies to R7.
- **R12** — LCSC Part # field is empty in the schematic (Task 1 finding); the correct matching
  LCSC ID (C95781) was identified via `get_jlcpcb_part` for reference but not written back to
  the schematic, since correcting sourcing data is outside this task's scope.
- **HC-TYPE-C-16P-01A (J3 on `usbc-programming-uart.kicad_sch`, the USB-C receptacle — a
  different J3 than the PowerPole connector above, on a different sheet)** — its `Datasheet`
  field is `https://www.usb.org/sites/default/files/documents/usb_type-c.zip`, the general USB-C
  **specification archive**, not a part-specific datasheet for this connector at all. Not
  localized (there's no single-part PDF to fetch — this is a wrong-reference finding, not a
  fetch failure). Worth a BOM-audit look at sourcing a genuine HCTL datasheet or distributor
  page for this connector; out of scope for this pass to fix.
