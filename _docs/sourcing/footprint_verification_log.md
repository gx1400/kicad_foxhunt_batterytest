# Footprint Verification Log

Generated/maintained by `_agent_tasks/footprint_verification.md`. Read-only audit — no
footprint, symbol, or schematic file is ever edited by this task. Anything scoring below
70% needs a human look before layout; see Findings below.

Method note: `get_jlcpcb_part`/the `integration` toolset was not invocable this run
(consistent with the other two recurring audits this session) — used direct
`get_footprint_info` calls against library footprints, plus WebFetch/WebSearch for
datasheet mechanical pages, instead.

## Findings (anything below 70%)

1. **F1 (Fuse_PTCC_CLM1612, Polytronics CLM1612P1412) — 45%.** The footprint has 3 pads
   of two different sizes (two 2.4×1.55mm pads + one 1.2×1.6mm pad, asymmetric), but
   LCSC's own listing describes CLM1612P1412 as a compact 3.2×1.6×0.65mm SMD part with no
   confirmed pin count. Polytronics' own datasheet couldn't be fetched (site unreachable,
   consistent with the datasheet-localization task's own finding for this same part) to
   settle whether this is genuinely a 3-terminal "current limiting module" (plausible —
   the part name itself suggests more than a plain 2-terminal PTC fuse) or a wrong
   footprint. Inconclusive, not confirmed wrong — needs the real datasheet in hand.
2. **L1, L2 (L_Chilisin_MHCI06030, MHCI06030-3R3M-R8) — 55%.** The footprint's own
   embedded description string cites a *different* Chilisin part number entirely
   (`BMRA00040415`, 4.6×4.1×1.5mm) instead of the real placed part. Fetched Chilisin's
   real MHCI06030-series mechanical spec directly: body is A=6.6±0.2mm × C=3.0mm max,
   noticeably bigger than the BMRA part the description names. The footprint's actual
   pad geometry (2.35×3.5mm pads, ~8.4mm outer span) is a plausible, generously-sized
   land pattern for the real 6.6×3.0mm-class part — so this is very likely fine
   electrically/mechanically, but the footprint's self-documentation names the wrong
   part, which is a real traceability bug (already flagged once before in
   `_docs/sourcing/3d_models.md`, still unresolved).
3. **BT3 (BAT-SMD_BS-08-B2AA016, actual MPN BS-08-B2AA020-R) — 60%.** The footprint file
   itself is named after a *sibling* MYOUNG part number (`...016`) that differs from the
   MPN actually in the schematic (`...020-R`). These look like same-family
   plating/packaging variants (MYOUNG's `BS-08-B2AA0xx` line), which often share an
   identical physical footprint, but that wasn't independently confirmed against a real
   MYOUNG mechanical drawing this pass — flagging the name mismatch for a human check
   rather than assuming they're identical.
4. **Fuse_MCCQ-122 (F2, Multicomp Pro MCCQ-122) — 65%.** Footprint has 4 physical pads
   but only 2 distinct pad numbers (two pads each tied to "1", two to "2") — plausible
   for a mini-blade fuseholder clip (twin prongs per terminal for grip strength), and
   internally consistent (duplicate-numbered pads share a net by KiCad convention), but
   Multicomp's own datasheet wasn't available to independently confirm this exact
   4-contact/2-net topology is correct for MCCQ-122 specifically.

## Log

| Manf | MPN | LCSC Part # | Refs | Package (declared/real) | Footprint used | Source | Cross-reference result | Confidence | Last audit |
|---|---|---|---|---|---|---|---|---|---|
| TI | SN74LVC1G08DBVR | C7666 | U4 | SOT-23-5 / SOT-23-5 | `Package_TO_SOT_SMD:SOT-23-5` | stock | Standard single-gate logic footprint, exact package match. | 98% | 2026-09-20 |
| HCTL | HC-TYPE-C-16P-01A | C2894897 | J3 | USB-C receptacle / same | `Connector_USB:USB_C_Receptacle_HCTL_HC-TYPE-C-16P-01A` | stock | Footprint literally named after this exact vendor+part number. | 98% | 2026-09-20 |
| JST | B3B-PH-SM4-TB(LF)(SN) | C160353 | J4,J5,J6,J7 | 3-pin PH / same | `Connector_JST:JST_PH_B3B-PH-SM4-TB_1x03-1MP_P2.00mm_Vertical` | stock | Footprint name is the exact JST part number. | 98% | 2026-09-20 |
| Phoenix Contact | MCV 1,5/2-GF-3,81 | — | J2 | 2-pos terminal block / same | `Connector_Phoenix_MC:PhoenixContact_MCV_1,5_2-GF-3.81_1x02_...` | stock | Footprint name is the exact Phoenix part number. | 97% | 2026-09-20 |
| Anderson Power Products | 1377G12-BK | — | J1 | PowerPole PCB-mount / same | `kicad_gx_library:Connector_Anderson_PP25_RA_1377g12-bk` | kicad_gx_library (hand-authored) | 2 THT pads, 7.9mm pitch, 1.8mm drill — close to Anderson's official 0.31"/7.874mm ganged-housing pitch (~0.03mm off). A real 3D model for this exact part was independently matched in this project's history (`3d_models.md`). Plausible and likely correct; not re-derived pin-by-pin from Anderson's own PCB dimension drawing this pass. | 80% | 2026-09-20 |
| MPD (Memory Protection Devices) | BH-18650-PC | C5339083 (delisted) | BT1,BT2 | 18650 holder / same | `Battery:BatteryHolder_MPD_BH-18650-PC` | stock | Stock KiCad footprint literally named after this exact manufacturer+MPN. | 96% | 2026-09-20 |
| muRata | GRM1885C1H103JA01D | C85973 | C1,C2,C3 | 0603 / same | `Capacitor_SMD:C_0603_1608Metric` | stock | Standard 0603 ceramic cap footprint, package matches MPN's own "1885" (0603 metric) code. | 97% | 2026-09-20 |
| muRata | GRM188R71H104KA93D | C77055 | C42,C50,C4,C5,C11,C12,C13,C18,C19,C24,C25,C26,C27,C44,C45,C46,C47,C48,C49,C51,C31,C33,C35,C37,C38,C39,C41,C30,C52 | 0603 / same | `Capacitor_SMD:C_0603_1608Metric` | stock | Standard 0603 ceramic cap footprint, matches "188" package code. | 97% | 2026-09-20 |
| muRata | GRM188R71A474KA61D | C435402 | C7,C8,C9 | 0603 / same | `Capacitor_SMD:C_0603_1608Metric` | stock | Same family as above. | 97% | 2026-09-20 |
| — (DNP) | — | — | C43 | n/a | `Capacitor_SMD:C_0603_1608Metric` | stock | Deliberate DNP placeholder, not a real sourced part — nothing to cross-reference. | n/a | 2026-09-20 |
| muRata | GCM219R71E474KA55D | C437481 | C6 | 0805 / same | `Capacitor_SMD:C_0805_2012Metric` | stock | Standard 0805 ceramic cap footprint, "219" = 0805 metric code. | 97% | 2026-09-20 |
| muRata | GRM21BZ71E106KE15L | C237493 | C14,C15,C16,C17,C34 | 0805 / same | `Capacitor_SMD:C_0805_2012Metric` | stock | Same family, "21B" = 0805 metric code. | 97% | 2026-09-20 |
| muRata | GRM21BR71H105KA12L | C77083 | C32,C36 | 0805 / same | `Capacitor_SMD:C_0805_2012Metric` | stock | Same family. | 97% | 2026-09-20 |
| muRata | GRM31CZ71C226ME15L | C909845 | C10,C20,C21,C22,C23 | 1206 / same | `Capacitor_SMD:C_1206_3216Metric` | stock | "31C" = 1206 metric code, matches. | 96% | 2026-09-20 |
| JARSON | XT1C220M0506 | C5260805 | C28,C29,C40 | SMD electrolytic / same | `Capacitor_SMD:C_Elec_5x5.8` | stock | MPN's own "0506" size code (5x6mm-class) matches the 5x5.8mm stock footprint reasonably closely. | 88% | 2026-09-20 |
| LUTE | 0805L050/16XR | C22435898 | F3 | 0805 fuse / same | `Fuse:Fuse_0805_2012Metric` | stock | MPN's own "0805" prefix matches the stock 0805 fuse footprint exactly. | 95% | 2026-09-20 |
| Kingbright | APT2012LZGCK | C5569446 | D7 | 0805 LED / same | `LED_SMD:LED_0805_2012Metric` | stock | MPN's "2012" = 0805 metric, matches. | 96% | 2026-09-20 |
| Kingbright | APTD2012LSURCK | C5366382 | D8 | 0805 LED / same | `LED_SMD:LED_0805_2012Metric` | stock | Same family. | 96% | 2026-09-20 |
| Kingbright | APTD2012LQBC/D | C5879058 | D9 | 0805 LED / same | `LED_SMD:LED_0805_2012Metric` | stock | Same family. | 96% | 2026-09-20 |
| Kingbright | APTD2012LSYCK | C5588998 | D10 | 0805 LED / same | `LED_SMD:LED_0805_2012Metric` | stock | Same family. | 96% | 2026-09-20 |
| Worldsemi | WS2812B-B/W | C114586 | D4 | PLCC-4 5.0x5.0mm / same | `LED_SMD:LED_WS2812B_PLCC4_5.0x5.0mm_P3.2mm` | stock | Stock footprint specifically named for this exact part family (WS2812B PLCC-4). | 97% | 2026-09-20 |
| Littelfuse | SMAJ18A | C148219 | D1 | SMA / same | `Diode_SMD:D_SMA` | stock | "SMAJ" prefix = SMA package, matches. | 96% | 2026-09-20 |
| YAGEO | RT0603BRE075KL | C862232 | R1,R4 | 0603 / same | `Resistor_SMD:R_0201_0603Metric` | stock | KiCad's dual-standard 0201/0603 footprint name; MPN's "0603" prefix confirms 0603. | 93% | 2026-09-20 |
| muRata | NCP15XH103F03RC | C77131 | TH1 | 0402 thermistor / same | `Resistor_SMD:R_0402_1005Metric` | stock | "XH" 0402-class Murata thermistor on the matching 0402 footprint. | 94% | 2026-09-20 |
| UNI-ROYAL | 0603WAF2372T5E | C22912 | R2,R3 | 0603 / same | `Resistor_SMD:R_0603_1608Metric` | stock | MPN's own "0603" prefix matches. | 96% | 2026-09-20 |
| YAGEO | RC0603FR-0730K1L, RT0603BRE0780K6L, RC0603FR-071ML, RT0603BRD0710KL, RT0603BRD0776K8L, RT0603BRD0752K3L, RC0603FR-07100KL, RC0603FR-07750RL, RC0603FR-07330RL, RC0603DR-07590RL, RC0603FR-071KL, RC0603JR-075K1L | (per-part, see Refs) | R5,R8,R7,R29,R18,R19,R30,R34,R32,R33,R20,R21,R31,R35,R36,R38,R37,R39,R40,R41,R42,R24,R25 | 0603 / same | `Resistor_SMD:R_0603_1608Metric` | stock | All YAGEO "0603" family MPNs on the matching 0603 footprint — consistent block, grouped here for brevity. | 96% | 2026-09-20 |
| YAGEO | RC0805FR-0710KL, RC0805FR-0710RL, RC0805FR-071KL, RC0805FR-07820RL | (per-part) | R6,R9,R16,R17,R26,R27,R28,R10,R14,R15 | 0805 / same | `Resistor_SMD:R_0805_2012Metric` | stock | All YAGEO "0805" family MPNs on the matching 0805 footprint. | 96% | 2026-09-20 |
| YAGEO | RT1206BRD07150RL | C870430 | R11,R12 | 1206 / same | `Resistor_SMD:R_1206_3216Metric` | stock | MPN's "1206" prefix matches. | 96% | 2026-09-20 |
| (SparkFun-catalog, no MPN/LCSC) | — | — | R22,R23 | 2W axial THT / same | `Resistor_THT:R_Axial_Power_L20.0mm_W6.4mm_P5.08mm_Vertical` | stock | Not JLCPCB-verifiable by design (standing BOM-audit exception) — generic parametric THT axial-power shape is a reasonable, standard choice for a 2W bleed resistor; can't cross-check against a real datasheet without an MPN/LCSC ID. | 75% | 2026-09-20 |
| Milliohm | HoJLR2512-3W-2.5mR-1% | C2904234 | R13 | 2512 4-pad Kelvin sense / same | `kicad_gx_library:RES-SMD_L6.4-W3.2-R2512_Sense4Pin` | kicad_gx_library (hand-authored) | Confirmed force pads 2.0×3.3mm + sense pads 0.5×0.5mm — matches this project's own prior direct verification against the Milliohm datasheet (`task1_lcsc_crossreference.md`). Re-confirmed, not re-derived from scratch. | 92% | 2026-09-20 |
| Multicomp Pro | MCCQ-122 | — | F2 | mini-blade fuseholder / plausible | `kicad_gx_library:Fuse_MCCQ-122` | kicad_gx_library (hand-authored) | See Findings #4 above. | 65% | 2026-09-20 |
| Polytronics (PTTC) | CLM1612P1412 | C5353611 | F1 | 3.2×1.6mm SMD / pin count unconfirmed | `kicad_gx_library:Fuse_PTCC_CLM1612` | kicad_gx_library (hand-authored) | See Findings #1 above. | 45% | 2026-09-20 |
| Chilisin | MHCI06030-3R3M-R8 | C108294 | L1,L2 | 6.6×3.0mm power inductor / footprint envelope plausible | `kicad_gx_library:L_Chilisin_MHCI06030` | kicad_gx_library (hand-authored) | See Findings #2 above. | 55% | 2026-09-20 |
| MYOUNG(美阳) | BS-08-B2AA020-R | C964787 | BT3 | CR2032 SMD holder / sibling-part footprint name | `kicad_gx_library:BAT-SMD_BS-08-B2AA016` | kicad_gx_library (easyeda2kicad) | See Findings #3 above. | 60% | 2026-09-20 |
| onsemi | BAS116LT1G | C232527 | D2,D3 | SOT-23 / same | `kicad_gx_library:SOT-23_L2.9-W1.3-P1.90-LS2.4-BR` | kicad_gx_library (easyeda2kicad) | Pad pitch is exactly 1.9mm — the real JEDEC SOT-23 (TO-236) standard — and the footprint's own name states it. Dimensionally correct standard SOT-23. | 92% | 2026-09-20 |
| onsemi | CAT24C32YI-GT3 | C94264 | U16 | TSSOP-8 / same | `kicad_gx_library:TSSOP-8_L4.4-W3.0-P0.65-LS6.4-BL` | kicad_gx_library (easyeda2kicad) | Pad pitch 0.65mm, body width 3.0mm — exact JEDEC MO-153 TSSOP-8 standard, matches the footprint's own name and the real part's confirmed TSSOP-8 package (verified via LCSC earlier this project). | 93% | 2026-09-20 |
| TI | TPS2121RUXR | C485916 | U1,U2,U3 | VQFN-12 2.5×2.0mm / same | `kicad_gx_library:VQFN-HR-12_L2.5-W2.0-P0.50-BL` | kicad_gx_library (easyeda2kicad) | Body envelope (2.5×2.0mm) and pitch (0.5mm) match this project's own already-researched TI package spec for this exact part (`ic_inventory.md`: "VQFN-12 (2.5×2.0mm)"). Pad-level detail (mixed side/thermal pad shapes) is plausible for TI's small "RUX" QFN variant but wasn't independently re-derived pin-by-pin from TI's own mechanical drawing this pass. | 80% | 2026-09-20 |
| ALJ | 8205A | C22458966 | Q2 | SOT-23-6 / same | `digikey-kicad-library:SOT23-6L` | other (vendored digikey-kicad-library, non-stock) | Standard 0.95mm-pitch SOT-23-6, footprint's own embedded description cites ST's generic SOT-23-6 mechanical reference (CD00047494) — a well-known industry-standard shape; 8205A is a common SOT-23-6 dual MOSFET. | 90% | 2026-09-20 |
| TI | PCA9555PWR | C2864778 | U15 | TSSOP-24 / same | `Package_SO:TSSOP-24_4.4x7.8mm_P0.65mm` | stock | Genuine stock KiCad symbol+footprint (`Interface_Expansion:PCA9555PW`), standard JEDEC TSSOP-24 pitch/body. Not a `kicad_gx_library` part despite initial suspicion. | 97% | 2026-09-20 |
| WCH | CH334R | C4154405 | U11 | QSOP-16 / same | `Package_SO:QSOP-16_3.9x4.9mm_P0.635mm` | stock | Standard QSOP-16, 0.635mm pitch matches JEDEC MO-137. | 95% | 2026-09-20 |
| WCH | CH340C | C84681 | U13 | SOIC-16 / same | `Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` | stock | Standard SOIC-16. | 96% | 2026-09-20 |
| NXP | PCF8563T/5,518 | C7440 | U14 | SOIC-8 / same | `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` | stock | Standard SOIC-8, matches NXP's real package. | 97% | 2026-09-20 |
| MAXIM (ADI) | MAX17320G22+ | C2914309 | U5 | TQFN-24-1EP 4x4mm EP2.1x2.1mm / same | `Package_DFN_QFN:TQFN-24-1EP_4x4mm_P0.5mm_EP2.1x2.1mm` | stock | Exact-EP-size match already independently confirmed in this project's history (`3d_models.md`). | 97% | 2026-09-20 |
| Espressif | ESP32-S3-WROOM-1-N8R8 | C2913201 | U10 | RF module / same | `RF_Module:ESP32-S3-WROOM-1` | stock | Stock KiCad footprint specifically for this exact module family. | 96% | 2026-09-20 |
| Yajingxin | TXM12M0004252DBCEO00T | C284154 | Y1 | SMD 2520 crystal / same | `Crystal:Crystal_SMD_2520-4Pin_2.5x2.0mm` | stock | Standard 2520 4-pad crystal footprint, plausible for a 12MHz SMD crystal of this class; not independently re-derived from Yajingxin's own drawing this pass. | 85% | 2026-09-20 |
| NDK | NX3215SA-32.768K-STD-MUA-9 | C519280 | Y2 | 3215 2-pad tuning fork / same | `Crystal:Crystal_SMD_3215-2Pin_3.2x1.5mm` | stock | NDK's own part-number prefix "NX3215" directly encodes the 3.2×1.5mm package — exact name-embedded match, this is NDK's textbook standard package for this part family. | 96% | 2026-09-20 |
| TI | LM1117IMPX-3.3/NOPB | C23984 | U9,U12 | SOT-223-3/4 / same | `Package_TO_SOT_SMD:SOT-223` | stock | 4-pad layout (3 small leads + 1 large tab) matches TI's own datasheet package table for the DCY/SOT-223 variant (independently read this project's history — "DCY (SOT-223, 4)", 6.50×3.50mm body). Already package-confirmed via a live TI datasheet read earlier this session. | 95% | 2026-09-20 |
| TI | LM1085IS-5.0RG | C2865978 | U8 | TO-263-3 / same | `Package_TO_SOT_SMD:TO-263-3_TabPin2` | stock | Exact-match 3D model already independently confirmed in this project's history (`3d_models.md`). | 96% | 2026-09-20 |
| TI | TPS563201DDCR | C116592 | U6,U7 | SOT-23-6 / same | `Package_TO_SOT_SMD:SOT-23-6` | stock | Standard, well-known TI buck-converter SOT-23-6 package. | 95% | 2026-09-20 |
| XKB Connection | DSHP04TS-S | C319050 | SW4 | 4-position DIP slide / same | `Button_Switch_SMD:SW_DIP_SPSTx04_Slide_KingTek_DSHP04TS_W7.62mm_P1.27mm` | stock | Footprint name is the exact KingTek part number this session already directly confirmed against (earlier conversation: "Does C319050 match with...DSHP04TS..." — confirmed yes). | 95% | 2026-09-20 |
| Korean Hroparts Elec | K2-1102SP-A4SC-04 | C83916 | SW1,SW2,SW3,SW7,SW8,SW9,SW10 | SMD tact switch / plausible generic match | `Button_Switch_SMD:SW_SPST_PTS645Sx43SMTR92` | stock | Uses the common "PTS645"-style generic tact-switch footprint name for a different manufacturer's (Korean Hroparts) compatible part — a very standard, widely-reused footprint convention for this switch shape, but not an exact vendor-name match like most other rows here. | 80% | 2026-09-20 |
| onsemi | MMBT2222ALT1G | C82460 | Q3,Q4 | SOT-23 / same | `Package_TO_SOT_SMD:SOT-23` | stock | Classic SOT-23 BJT, standard. | 95% | 2026-09-20 |
| AOS | AO3407A | C15155 | Q5,Q8 | SOT-23 / same | `Package_TO_SOT_SMD:SOT-23` | stock | Standard SOT-23 MOSFET package. | 94% | 2026-09-20 |
| AOS | AO3400A | C20917 | Q1 | SOT-23 / same | `Package_TO_SOT_SMD:SOT-23` | stock | Same family. | 94% | 2026-09-20 |
| onsemi | BSS138LT1G | C82045 | Q6,Q7 | SOT-23 / same | `Package_TO_SOT_SMD:SOT-23` | stock | Package directly confirmed via LCSC earlier this session (SOT-23). | 96% | 2026-09-20 |
