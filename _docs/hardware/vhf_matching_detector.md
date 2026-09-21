# VHF Antenna Matching Network & RF Forward-Power Detector — Parts & Layout

Covers two RF circuits that sit between the SA818S-V (`U19`) and the antenna SMA
connector (`J10`-style, Amphenol 132289): the **7-element low-pass matching network**
(topology borrowed from the open-source [KV4P-HT v2.0e](https://github.com/VanceVagell/kv4p-ht)
reference design) and the **diode-detector forward-power sensing tap** feeding
`esp32s3_pinout.md`'s `IO4` (ADC) / `IO41` (comparator fault flag) pair, per
`controller_platform.md` § RF power sensing. Neither circuit is in the schematic yet —
this is the sourcing/topology plan for the next placement pass.

**Rigor note, read before ordering:** both circuits have values that are *starting
points*, not final — the matching network's element values come from a reference
design at a different board's layout/stray capacitance, and the detector's coupling
ratio depends on parasitics that only a real bench measurement will pin down. Treat
every value here as "populate this, then verify/tune on the bench," the same way this
project's mic-level attenuator trimmer (`audio_ptt_path.md`) needed a real deviation
measurement before the calculated nominal was trusted.

**Package size: 0805 throughout, with one deliberate exception.** Bench tuning means
repeated hand rework, so everything below is spec'd in 0805 rather than 0402 for
easier soldering/swapping. At 145MHz (λ ≈ 2.07m) an 0805 footprint is still a tiny
fraction of a wavelength, so this doesn't introduce a meaningful parasitic penalty —
ESL/ESR/SRF differences between 0402 and 0805 at this frequency are noise next to the
tolerances this whole design already assumes need bench tuning. **The one exception:
`C_couple` (1pF) in the detector circuit stays at 0402.** It's deliberately a tiny
value to set the RF sample ratio, and an 0805 pad's own parasitic capacitance-to-ground
is large enough *relative to 1pF* to meaningfully shift that ratio — the one part here
where package size is a real, not just theoretical, concern.

## 1. VHF matching network — SA818S `ANT` → `J10` (SMA)

7-element LC low-pass ladder, 4 shunt capacitors + 3 series inductors, tuned for
145MHz (2m VHF). Topology and starting values carried over from KV4P-HT v2.0e's own
antenna output stage (a real, fielded ESP32+SA818/DRA818 design), rendered and
inspected directly from its schematic earlier this project.

```
 U19            C1        L1        C2        L2        C3        L3        C4        J10
 ANT  o──────────┬────///────┬────///────┬────///────┬────///────┬──────────────o SMA (50Ω)
                 │    27nH   │    30nH   │    43nH*   │           │
               ──┴──       ──┴──       ──┴──       ──┴──         │
               10pF        18pF        16pF        5.1pF         │
                 │           │           │           │           │
                GND         GND         GND         GND         GND
```

*(ladder read left→right: shunt C1, series L1, shunt C2, series L2, shunt C3,
series L3, shunt C4 — 4 shunt + 3 series = 7 elements)*

| Ref (proposed) | Function | Value | MPN | Package | LCSC | Stock | Notes |
|---|---|---|---|---|---|---|---|
| C1 | Shunt | 10pF | YAGEO CC0805JRNPO9BN100 | 0805, C0G/NP0, 50V | `C107107` | 515,400 | Verified in stock. |
| L1 | Series | 27nH | Murata LQW2BAS-class (0805) | 0805, wirewound, ±5% | *unconfirmed* | — | Murata's `LQW2BAS`/`LQW2BH` 0805 wirewound families cover this value (2.7–820nH range, real product lines) — **exact LCSC C-number for the 27nH variant not confirmed this pass**; search LCSC directly before ordering. |
| C2 | Shunt | 18pF | YAGEO CC0805JRNPO9BN180 | 0805, C0G/NP0, 50V | *unconfirmed* | — | Real Yageo catalog part (multiple distributors confirm it) — **LCSC C-number not confirmed this pass**. |
| L2 | Series | 30nH | Murata LQW2BH-class (0805) | 0805, wirewound, ±5% | *unconfirmed* | — | Same family as L1 — value is within Murata's real 0805 product range, C-number not confirmed this pass. |
| C3 | Shunt | 16pF | YAGEO CC0805JRNPO9BN160 | 0805, C0G/NP0, 50V | *unconfirmed* | — | Real Yageo catalog part — **LCSC C-number not confirmed this pass**; search LCSC directly for `CC0805JRNPO9BN160`. |
| L3 | Series | 43nH* | *(no direct LCSC match found)* | 0805 | — | See note below — recommend substituting **47nH** (below) unless exact 43nH is bench-confirmed necessary. |
| C4 | Shunt | 5.1pF | YAGEO CC0805-class NP0 | 0805, C0G/NP0, 50V | *unconfirmed* | — | Real Yageo value/series — **LCSC C-number not confirmed this pass**. |

**L3 (43nH) substitution note:** no genuine 43nH chip inductor (0402 or 0805) turned up
on LCSC this pass. Coilcraft makes real, purpose-built high-Q parts at exactly 43nH
(`0402CS-43N`, `0402DC-43N`, `0402HP-43N`, and 0805-class equivalents) but Coilcraft
isn't an LCSC-stocked brand — sourcing those means going outside the normal JLCPCB
assembly flow, the same category as the SA818S-V module itself (`ic_inventory.md`'s
"DNP — sourced outside LCSC"). The practical alternative: **Murata LQW15AN47NJ00D**
(47nH, ±5%, 0402, LCSC `C192855`, in stock) is a real, in-stock, confirmed substitute —
about 9% off the reference design's 43nH, well within what a network that already needs
bench retuning can absorb. **This one is worth keeping at 0402** rather than chasing an
unconfirmed 0805 47nH part, since it's the one inductor position with a fully verified
LCSC listing already — a bench-tuning board can mix package sizes on a single part
without issue; populate 47nH unless the bench tuning pass specifically calls for the
exact reference value.

**Inductor sourcing gap, general note:** unlike the caps (Yageo's catalog is
comprehensive and every value above is a genuine, if not always LCSC-confirmed, part),
0805 RF inductors at these exact odd values (27nH, 30nH) didn't turn up specific LCSC
C-numbers this pass, only confirmation that Murata's real 0805 wirewound families cover
the range. Confirm exact part numbers on LCSC directly before ordering, or keep L1/L2 at
their already-fully-verified 0402 listings (`C113112`, `C329622` from the original pass)
if the 0805 lookup doesn't pan out — same "mixed package sizes are fine" reasoning as L3.

## 2. RF forward-power detector — tap after the matching network

Simple diode envelope detector, forward-power only (no reflected/SWR), per
`controller_platform.md`. Taps the antenna line *after* C4/L3 (closer to the actual
radiated signal than the SA818's raw unfiltered PA output), rectifies a small sampled
fraction into a DC level, and feeds both the ADC readback (`IO4`) and the comparator
fault flag (`IO41`, via `U`-tbd LM393).

```
                    C_couple                                    R_load      C_filter
 (after C4/L3) o──────||──────┬── R_series ──▶|── D1 ──┬───────/\/\/\──┬──────||──────┐
   ANT node         1pF       │      470Ω    (anode)   │      100kΩ    │    1nF       │
                               │                        │               │              │
                            C_shunt                     └── detector DC node ─────────┤
                             100pF                            │              │        │
                               │                               ▼              ▼        │
                              GND                         → IO4 (ADC1)   → LM393 (+)  GND
                                                                            │
                                                                     LM393 (−) ← 3.3V ÷ R_ref1/R_ref2
                                                                            │
                                                                            ▼
                                                                     → IO41 (digital fault flag)
```

| Ref (proposed) | Function | Value | MPN | Package | LCSC | Stock | Notes |
|---|---|---|---|---|---|---|---|
| C_couple | RF sample tap (series) | 1pF | YAGEO CC0402CRNPO9BN1R0 | **0402** (kept small — see note above), C0G/NP0, 50V, ±0.25pF | *unconfirmed* | — | Real Yageo part (Farnell/DigiKey/Mouser/RS all list it) — **LCSC C-number not confirmed this pass**; verify directly on LCSC before ordering. **Deliberately not respec'd to 0805** — see the package-size note at the top of this doc. |
| C_shunt | Divider reference (shunt) | 100pF | YAGEO CC0805JRNPO9BN101 | 0805, C0G/NP0, 50V | `C62768` | 630,100 | Sets the coupling ratio with `C_couple` (~1:100, ≈ -40dB) — **starting point only, needs bench tuning** against real measured TX power. |
| R_series | Diode protection/linearization | 470Ω | YAGEO RC0805FR-07470RL | 0805, thick film, ±1% | `C114564` | Verified listed | Same standard 470Ω already used elsewhere on this board (`R47`/`R48`, DAC output filters, though those are likely 0603 — reuse whatever MPN those turn out to already be sourced as, for consistency, if it matters more than the package-size goal here). |
| D1 | Envelope rectifier | — | High Diode BAT54 (or PC817C-S-class Schottky, single) | SOT-23 (discrete diode — package respec doesn't apply the same way) | `C466635` | 2,000 | Small-signal Schottky, not a specialized zero-bias RF part — deliberate: this circuit samples watts of real TX power, not microwatts, so a general-purpose Schottky (fast, low Vf) is the right fit, not an RFID-grade `HSMS-2850` (not LCSC-stocked anyway, and rated for signals far weaker than what's available here). Modest LCSC stock — confirm at build time. |
| R_load | Detector bleed/load | 100kΩ | YAGEO RC0805FR-07100KL | 0805, thick film, ±1% | `C96346` | 787,600 | Sets the RC decay time with `C_filter`. |
| C_filter | Envelope/lowpass filter | 1nF | YAGEO CC0805JRX7R9BB102 | 0805, X7R, 50V | `C277515` (unconfirmed vs. alt listing) | ~2,700 | Modest stock for an X7R part at this common value — worth double-checking at build time. X7R is fine here (this cap is doing DC/envelope filtering, not RF-critical), no need for the NP0 precision the matching network caps want. |
| R_ref1, R_ref2 | Comparator threshold divider | 10kΩ / 10kΩ (start) | YAGEO RC0805FR-0710KL | 0805, thick film, ±1% | `C84376` | Verified listed | Symmetric divider gives a ~1.65V starting threshold off `+3.3V` — **exact trip point is a bench-tuning decision** (per `controller_platform.md`, "exact values TBD during detector circuit design"), not a calculated final value. May end up asymmetric once a real "antenna disconnected" DC level is measured. |
| U (comparator) | Fault-flag comparator | — | onsemi LM393DR2G | SOIC-8 (IC — package respec doesn't apply) | `C7955` | 209,190 | Already the project's selected part for this role — see `ic_inventory.md` § Forward-Power Fault Comparator. Open-drain output needs its own pull-up to `+3.3V` before `IO41`. |

**Optional protection**, not yet in the table above: a small clamp (zener or resistor
sized to limit worst-case voltage) at the `IO4` ADC tap, cheap insurance against an
under-tuned coupling ratio overdriving the ESP32-S3's ADC input during bring-up —
worth adding once the divider's real DC output range is bench-measured.

## 3. Open items

- [ ] Confirm LCSC C-numbers for the 0805 caps flagged `*unconfirmed*` above (`C1`
  10pF's 0805 variant is confirmed; `C2` 18pF, `C3` 16pF, `C4` 5.1pF, and `C_couple`'s
  0402 part are real Yageo catalog parts but not confirmed against LCSC's own listing
  this pass) and for `L1`/`L2`'s 0805 Murata inductors — search LCSC directly rather
  than trusting an assumed number.
- [ ] Decide on `L3`: populate the confirmed 47nH substitute (`C192855`, 0402) or source
  genuine 43nH (Coilcraft, outside LCSC) — bench tuning may make this moot either way.
  This one's deliberately staying 0402 since it's the fully-verified listing.
- [ ] Bench-tune the actual matching network once populated (real VNA/return-loss
  measurement, not just the calculated ladder) — same caveat KV4P-HT's own design
  would have needed on a different board.
- [ ] Bench-tune the detector's coupling ratio (`C_couple`/`C_shunt`) and comparator
  threshold (`R_ref1`/`R_ref2`) against real measured TX power once both circuits are
  populated — this doc's values are calculated starting points, not final.
- [ ] Confirm `R47`/`R48`'s actual sourced 470Ω MPN (`audio_ptt_path.md`) and reuse it
  for `R_series` here if different from `RC0603JR-07470RL`, to avoid two part numbers
  for the same standard value on one board.

## 4. Reference documents

- [`controller_platform.md`](controller_platform.md) § RF power sensing — the
  ADC/comparator dual-path design this detector feeds.
- [`sa818_pinout.md`](sa818_pinout.md) — `ANT` pin, and the shared Amphenol 132289
  SMA connector choice.
- [`esp32s3_pinout.md`](esp32s3_pinout.md) — `IO4` (ADC1_CH3) and `IO41` (MTDI) pin
  assignments.
- [`ic_inventory.md`](ic_inventory.md) § Forward-Power Fault Comparator — `LM393DR2G`
  sourcing detail.
- [KV4P-HT v2.0e](https://github.com/VanceVagell/kv4p-ht) — real, fielded open-source
  ESP32+SA818/DRA818 project this matching network's topology and starting values are
  drawn from.
