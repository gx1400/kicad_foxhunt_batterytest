# 3D models — search & download status

Task 4 deliverable. Scope: custom (non-stock) footprints used across `kicad_gx_library` and the
project's libraries. Download only — no linking/alignment, per the task's explicit instruction
(that's a manual step).

**Format policy (updated): STEP (.stp/.step) is priority, IGES (.igs/.iges) is the only
acceptable fallback. VRML (.wrl) is explicitly excluded — not downloaded even when it's the
only thing available.**

## Already covered, no action needed

| Part | Footprint | Status |
|---|---|---|
| Anderson Powerpole connector (J3, 1377G12-BK) | `kicad_gx_library:Connector_Anderson_PP25_RA_1377g12-bk` | **Already present**, and already compliant with the STEP/IGES policy — `libs/kicad_gx_library/3dmodels/GX.3dshapes/Anderson_PP25_1377G12_Housing.igs` and `Anderson_PP_15-45_Std.step`. Footprint doesn't reference them yet (manual linking step, out of scope here). |
| Chilisin inductor (L3/L4, MHCI06030-3R3M-R8) | `kicad_gx_library:L_Chilisin_MHCI06030` | **Already linked** to a stock KiCad model (`.step`) — footprint's own `(model ...)` block points at `${KICAD10_3DMODEL_DIR}/Inductor_SMD.3dshapes/L_Chilisin_BMRA00040415.step`. Compliant, nothing to do. |
| 8205A dual-FET (Q2) | `digikey-kicad-library:SOT23-6L` | Standard SOT-23-6 package — **`/usr/share/kicad/3dmodels/Package_TO_SOT_SMD.3dshapes/SOT-23-6.step`** confirmed present on disk. Compliant, nothing to do. |
| MAX17320 (U2, QFN-24-1EP 4x4mm EP2.1x2.1mm) | `Package_DFN_QFN:TQFN-24-1EP_4x4mm_P0.5mm_EP2.1x2.1mm` | **Verified present**: `/usr/share/kicad/3dmodels/Package_DFN_QFN.3dshapes/Texas_RGE0024C_VQFN-24-1EP_4x4mm_P0.5mm_EP2.1x2.1mm.step` (exact EP-size match) and a generic `HVQFN-24-1EP_4x4mm_P0.5mm_EP2.1x2.1mm.step` in the same folder. |
| LM74610 (U7/U10, VSSOP-8 3x3mm) | `Package_SO:VSSOP-8_3x3mm_P0.65mm` | **Verified present**: `/usr/share/kicad/3dmodels/Package_SO.3dshapes/VSSOP-8_3x3mm_P0.65mm.step` — exact match. |
| TPS563201 (U1/U3, SOT-23-6) | `Package_TO_SOT_SMD:SOT-23-6` | Same stock model as the 8205A entry above — already covered. |
| AMS1117-3.3 (U9, SOT-223-3) | `Package_TO_SOT_SMD:SOT-223-3_TabPin2` | **Verified present**: `/usr/share/kicad/3dmodels/Package_TO_SOT_SMD.3dshapes/SOT-223.step` — generic 3-pin SOT-223 shape, correct stand-in. |
| LM1085-5.0 (U8, TO-263-3) | `Package_TO_SOT_SMD:TO-263-3_TabPin2` | **Verified present**: `/usr/share/kicad/3dmodels/Package_TO_SOT_SMD.3dshapes/TO-263-3_TabPin2.step` — exact match. |

All 5 standard-package ICs from the original task list are confirmed (file paths checked directly
on disk, not just assumed) to already have compliant STEP models bundled with the stock KiCad 10
install. No download action needed for any of them.

## Genuinely missing — searched again, still gated/unreachable

| Part | Footprint | Sources tried this pass | Result |
|---|---|---|---|
| Polytronics CLM1612P1412 PTC fuse (F2) | `kicad_gx_library:Fuse_PTCC_CLM1612` | Mouser product page (timeout), Polytronics' own site `polytronics.com.tw` (connection refused — site appears unreachable from this environment) | Not found. Previously also checked: LCSC/EasyEDA (requires JS session or a conversion tool like `easyeda2kicad`, no plain download link). |
| Multicomp Pro MCCQ-122 mini-blade fuseholder (F3) | `kicad_gx_library:Fuse_MCCQ-122` | Digikey product page (HTTP 403 Forbidden), Farnell datasheet page (timeout) | Not found. Previously also checked: Component Search Engine, SnapEDA, Ultra Librarian — all require account sign-in or are JS-rendered with no static download link. |

Both parts have real STEP/IGES models that exist somewhere (confirmed by name/listing on the
gated sites), but no plain, unauthenticated `curl`-able URL was found across two search passes
and 8+ distinct sources. A human can retrieve either in under a minute via a browser session
(the Component Search Engine link for MCCQ-122, and Mouser's product page for the Polytronics
fuse, are both quick manual downloads) — that's the practical path forward rather than further
automated attempts.

## Recommendation

For the two remaining gated fuse models: download manually via a browser (links above), or set up
`easyeda2kicad` (a real, actively maintained CLI tool) as a one-time integration if this project
ends up needing many more EasyEDA/LCSC-sourced 3D models going forward.

## Uncommitted state

No git commits made. Files present on disk from this pass: none newly downloaded (everything
in scope was either already present/linked, or remains gated after a genuine retry). The 5
standard-IC entries above are newly *verified* (file existence confirmed) but were not new
downloads — they were already part of the stock KiCad install.
