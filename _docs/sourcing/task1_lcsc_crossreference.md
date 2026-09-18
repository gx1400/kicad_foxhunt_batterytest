# Task 1 — LCSC / MPN / Manufacturer Cross-Reference

Verification method: every component's recorded `LCSC Part #` field was looked up against the
real JLCPCB catalog (`get_jlcpcb_part`), and the returned MPN + manufacturer compared against
what's recorded in the schematic's own `MPN`/`Manf` fields. 26 unique LCSC part numbers cover
54 of the 126 placed component instances (the rest are power symbols, jumpers, test points, and
a handful of non-LCSC-sourced parts — not applicable to this method).

**Result: 25 of 26 verify cleanly** (MPN + manufacturer match exactly, allowing standard
company-name abbreviations — TI/Texas Instruments, muRata/Murata Electronics, AOS/Alpha & Omega
Semiconductor, MAXIM/Analog Devices-Maxim).

## Convention note

**Capacitor voltage ratings in the `Value` field are a minimum requirement, not the exact rating
of the sourced part.** A part rated higher than the labeled value (e.g. a 50V-rated part used
where the schematic says "25V") is expected and correct — it's not a defect and shouldn't be
flagged as a mismatch in future review passes. This is why the checklist below does *not* list
the 10 bypass-cap instances (C1,C2,C7,C8,C11,C15,C32,C33,C34,C35 — labeled `0.1u/25V`, actually
sourced as a 50V-rated part) as something to fix; instead there's a task to document this
convention directly on each sheet so it doesn't get re-flagged later.

## Checklist

- [ ] **U2 (MAX17320): blank `Value` field.** Set to `MAX17320` — this was already fixed once on
      the old `U1` reference earlier in the project, and appears to have been lost during the
      hierarchy restructure/renumbering to U2.
- [ ] **R11: footprint names a manufacturer that doesn't match the sourced part.** Footprint is
      `R_Shunt_Vishay_WSK2512_6332Metric_T2.21mm` (implies Vishay WSK2512 pad geometry), but the
      recorded/sourced MPN is `HoJLR2512-3W-2.5mR-1%` by **Milliohm**, a different manufacturer.
      Verify Milliohm's actual pad dimensions match the Vishay-based footprint before fab — if
      they differ even slightly this is a real physical footprint mismatch, not just naming.
- [ ] **U8: sourced part is a clone, not the original.** `LM1085IS-5.0RG` by HANSCHIP
      semiconductor is a second-source part, not the original TI/onsemi LM1085. Probably fine
      (same function/pinout is typical for these), but confirm datasheet specs (dropout, ESR
      stability requirements per README §3) actually match before relying on it.
- [ ] **BT1: LCSC Part # (`C5339083`) not found in JLCPCB's current catalog.** Delisted or stale —
      verify on LCSC.com directly, or re-source.
- [ ] **BT2: missing LCSC Part # entirely** (BT1, the same physical part, has one — see above).
      Copy over once BT1 is resolved.
- [ ] **R12: has an MPN (`RC0805FR-071KL`, Yageo) but no LCSC Part #.** Fillable — looks like a
      real, findable Yageo part.
- [ ] **R16, R7 (2W THT resistors): no MPN/Manf/LCSC at all**, just a SparkFun catalog ID
      (`PROD_ID`). Not JLCPCB-verifiable as recorded — fine if intentionally SparkFun-sourced,
      otherwise needs real sourcing data.
- [ ] **Add a sheet note documenting the capacitor-voltage-is-a-minimum convention** (see above),
      so it isn't mistaken for a defect in a future review:
  - [ ] `pcb/foxhunt1.kicad_sch`
  - [ ] `battery_18650_input.kicad_sch`
  - [ ] `battery_powerpole_input.kicad_sch`
  - [ ] `power_regulation.kicad_sch`

## Informational — no action needed

- **J3, J4, F3** are sourced from Digikey/Newark/Mouser rather than LCSC — not verifiable against
  the JLCPCB catalog by this method, but no internal inconsistency found in their own recorded
  fields (MPN/Manf are self-consistent).
- **25 of 26 LCSC-sourced parts verified clean** — see the full unique-part list and verification
  detail in the conversation; not duplicated here since nothing needs fixing on them.
