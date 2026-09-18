# Project Library

Local library resources for parts used in this project that are **not** part of the shared
`libs/kicad_gx_library/` (which holds reusable custom symbols/footprints/3D models
maintained across projects). Everything here is specific to `foxhunt1`.

## Structure

- `datasheets/` — PDF datasheets for placed components, organized into per-manufacturer
  subfolders (e.g. `Murata_Electronics/`, `YAGEO/`, `TI/`). Schematic component `Datasheet`
  fields reference these files via `${KIPRJMOD}`-relative paths. See
  `_docs/project_library_datasheets.md` for the full part-to-file mapping, including two
  parts (BH-18650-PC, 1377G12-BK) with no available datasheet source.
- `symbols/` — reserved for project-specific KiCad symbols not in `kicad_gx_library`.
  Not yet populated.
- `footprints/` — reserved for project-specific KiCad footprints not in `kicad_gx_library`.
  Not yet populated.
- `3dmodels/` — downloaded 3D models (STEP preferred) for manual placement/alignment.
  Not linked to schematic/footprint parts automatically — see
  `_docs/project_library_3d_models.md` once populated.
