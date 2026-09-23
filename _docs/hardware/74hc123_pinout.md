# 74HC123 — Dual Retriggerable Monostable — Pinout & Planned Interconnects

Covers the dual retriggerable monostable multivibrator (Nexperia **74HC123D,653**, LCSC
`C5597`, SOIC-16/SOT109-1 — see `ic_inventory.md`) with **two independent planned uses on
this board**: one section as the TX-safety PTT hardware timeout (`controller_platform.md`
§ TX safety), the other — the "spare half" — as the hardware force-off failsafe
(`open_items.md`). KiCad symbol `74xx:74HC123` / footprint
`Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` (stock libraries, both pin-verified against
Nexperia's own datasheet — no library fix needed).

**Placement status, corrected 2026-09-23:** the part (`U21`) is now placed and substantially
wired on `SA818V.kicad_sch` — the note below about this being purely a planning document is
stale for unit A specifically (RC timing, power-up circuit, and the NAND/switch output chain
are built — see § 3 and the pin table). Still genuinely unbuilt: unit B (force-off failsafe)
entirely, and unit A's MCU-retrigger path (§ 3's open item). Everything else below retains
its original framing: real pin functions from the datasheet, but "planned tie/connection" is
intent pulled from `controller_platform.md` / `open_items.md` / `audio_ptt_path.md` except
where a section explicitly says otherwise. Two of the four trigger nets involved (the
retrigger source for the TX-timeout half, and the level of `POWER_PB_SIGNAL` feeding the
force-off half) depend on design decisions that aren't made yet either — flagged
individually below, and again in § 4.

## 1. Full pin table

Reference: Nexperia `74HC_HCT123` datasheet Rev. 13 (2024-02-21), Table 2 (pin
description) and Table 3 (function table), SO16/SOT109-1 pinout.

| Pin | Name | I/O | Function (per Nexperia's own datasheet) | Planned tie/connection |
|---|---|---|---|---|
| 1 | 1A | I | Negative-edge triggered input, unit A (Schmitt) | **Unit A = TX-safety timeout.** Not yet decided whether A or B is the active retrigger input for this half — see § 3. If unused, tie to the level the function table calls for with the other input active (function table: triggering needs `nRD`=H; with `nB`=H, a falling edge on `nA` triggers — so if B is the active input instead, tie `1A` low). |
| 2 | 1B | I | Positive-edge triggered input, unit A (Schmitt) | Mutual with `1A` above — whichever of A/B carries the periodic MCU "keep transmitting" pulse from the PTT-intent path; the other ties to its inactive level. |
| 3 | 1R̄D | I | Direct reset LOW (async, overrides everything) + positive-edge triggered input | **Built** (2026-09-23): Nexperia's own Fig. 12 power-up circuit, plus a fast-discharge diode — see § 3.1. No "kill PTT now" system-level reset source feeds this pin; it's purely the power-up-glitch guard. |
| 4 | 1Q̄ | O | Active-LOW output, unit A | Planned, with `13`/`1Q`: series-gates the existing PTT path (`IO16` → `Q10` → `J11` → SA818S/HT `PTT`, see `sa818_pinout.md`/`audio_ptt_path.md`) per `controller_platform.md` § TX safety — "gates the PTT line in series with the MCU's own PTT-intent GPIO." Exact insertion point (ahead of `Q10`'s gate, in series with its drain, or ahead of the still-unselected PTT-keying optocoupler) is undecided — open item. |
| 13 | 1Q | O | Active-HIGH output, unit A | See `1Q̄` above — whichever polarity fits the eventual gate implementation. |
| 14 | 1CEXT | — | External timing capacitor connection, unit A | RC timing network for the **10-second hard TX ceiling** — see § 3 for the populated values (`C80`). Per the datasheet's own app-note (Fig. 11): cap goes between this pin and `15`/`1REXT/CEXT`. |
| 15 | 1REXT/CEXT | — | External timing resistor+capacitor connection, unit A | Resistor to VCC, per the same app-note figure. See § 3 for values. |
| 5 | 2Q | O | Active-HIGH output, unit B | **Unit B = spare-half hardware force-off failsafe** (`open_items.md`). Planned to assert an override of `PWR_LATCH`/`Q6`'s hold (`controller_platform.md` § Power sequencing) if the software hold-to-power-off timeout is exceeded. The actual override mechanism — how a one-shot output edge forces `PWR_LATCH` low against a possibly-still-asserting `Q6` — is the main undecided piece here, not just an RC value. |
| 6 | 2CEXT | — | External timing capacitor connection, unit B | RC target: "longer than the software hold threshold" (`open_items.md`). That firmware threshold isn't documented yet (hold-to-power-off firmware is itself still TBD per the same doc), so no concrete value can be calculated for this half yet — blocked on a firmware constant, unlike unit A's timing. |
| 7 | 2REXT/CEXT | — | External timing resistor+capacitor connection, unit B | Same block as `2CEXT` above. |
| 8 | GND | — | Ground | Planned: `GND`. |
| 9 | 2A | I | Negative-edge triggered input, unit B (Schmitt) | Planned force-off trigger, per `open_items.md`'s leading option: "triggered directly by SW3 (or its isolated `POWER_PB_SIGNAL` node)." **See the voltage-domain flag in § 4 — this can't actually be direct.** |
| 10 | 2B | I | Positive-edge triggered input, unit B (Schmitt) | Mutual with `2A` — same flag applies. |
| 11 | 2R̄D | I | Direct reset LOW (async) + positive-edge triggered input, unit B | **Currently hard-tied to `GND`** (confirmed against the live schematic, 2026-09-22 circuit review) — the opposite of `1R̄D`'s "inactive" tie: this holds unit B in permanent reset, a safe placeholder since it isn't built yet, not a step toward the Fig. 12 power-up RC. Once unit B's design is real, this should get its own copy of `1R̄D`'s power-up-RC + diode treatment (§ 3.1) instead of the hard GND tie. Also the natural place to let firmware abort the hardware failsafe once a normal software power-off succeeds in time, if that's ever wanted — not yet decided. |
| 12 | 2Q̄ | O | Active-LOW output, unit B | See `2Q` above. |
| 16 | VCC | — | Supply, 2.0–6.0V (74HC) | Planned: not yet decided which rail — see § 4. |

## 2. Trigger logic (datasheet Table 3, function table)

For either unit, with `nR̄D` = HIGH: a falling edge on `nA` while `nB` is HIGH triggers a
new pulse, or a rising edge on `nB` while `nA` is LOW triggers a new pulse — either
re-triggers (extends) an already-running pulse the same way. `nR̄D` = LOW forces `nQ` LOW /
`nQ̄` HIGH immediately regardless of `nA`/`nB`, overriding any in-progress pulse. This is
what makes it *retriggerable*: as long as the active input keeps getting valid edges before
the RC-timed pulse would otherwise end, the output stays asserted indefinitely — exactly
the property the TX-safety design in `controller_platform.md` depends on ("the MCU must
periodically retrigger it during a legitimate transmission").

## 3. Unit A — TX-safety timeout: RC values

Target: **10-second hard ceiling** per `controller_platform.md` § TX safety (revised down
from an earlier 2-minute target — 2026-09-22).

Datasheet's own pulse-width formula (§10, footnote [2], valid for `CEXT` > 10nF):

```
tW (ns) = K × REXT (kΩ) × CEXT (pF)
K ≈ 0.45 at VCC = 5.0V, ≈ 0.55 at VCC = 2.0V (Fig. 6 — K drifts with VCC, "typical" only)
```

**Populated**: `R60` = 470kΩ (`REXT`), `C80` = 47µF (`CEXT`) — confirmed against the live
`SA818V.kicad_sch`. At K = 0.45: `tW ≈ 0.45 × 470 × 47,000,000 ns ≈ 9.94s`, close enough to
the 10s target given `K`'s own typical-only characterization (see rigor note below).

**Rigor note, same as this board's other tuned RC/LC values (matching network, mic
attenuator):** this is a calculated figure from the datasheet's typical-K formula, not a
bench-verified number — `K` itself is only characterized at the 2V/6V endpoints in the
datasheet (Fig. 6), and electrolytic/tantalum tolerance at these values is loose. Verify
the actual timeout against a bench timer before trusting it for the real 10s ceiling. Per
the datasheet's own app-note (Fig. 11): `CEXT` connects between pins 14/6 and 15/7, `REXT`
from pin 15/7 to VCC — here `C80` bridges pin 14's own GND tie back up to the `RCEXT` (pin
15) node rather than running point-to-point between the two pins directly; electrically
equivalent since pin 14 has no other component between it and that GND tie.

Also relevant to the retrigger-pulse firmware: the datasheet gives a **minimum retrigger
interval** formula (§10, footnote [3]) — `t_rtrig ≈ 30 + 0.19×REXT×CEXT^0.9 +
13×REXT^1.05` ns (REXT in kΩ, CEXT in pF) — the MCU's periodic "still transmitting" pulses
need to be spaced at least this far apart to reliably retrigger rather than being missed;
at the populated values this works out to roughly **0.7s**, still far below any sane
firmware polling interval, but it's a real constraint worth having in the design record.

**Open item, not yet wired:** despite the values above being populated, the actual
retrigger path isn't built yet — see § 4's stale-cross-reference note and the live-wiring
caveat below. `1A` (pin 1) is currently driven by an inverted, un-pulsed `SA818_IN_PTT`
(via `U23`, a NAND-as-inverter), so the timeout is really a one-shot hard-cutoff from
key-down rather than an MCU-retriggered watchdog — the periodic heartbeat this section's
`t_rtrig` figure is meant to bound (`MCU_OUT_PTT_HB` → `SA818_IN_PTT_HB`) reaches this
sheet but dead-ends at its own pull-down (`R65`), never reaching `U21`. Also, `1B` (pin 2)
is currently floating — two wire stubs run toward a nearby `+3.3V` symbol and stop short.
Both need to be finished before this matches the retriggerable-watchdog design intent
described in `controller_platform.md` § TX safety.

### 3.1. Unit A — power-up glitch guard on `1R̄D` (pin 3) — built, 2026-09-23

Nexperia's own datasheet, §11.2 "Power-up considerations," Fig. 12: *"When the monostable is
powered-up it may produce an output pulse... This output pulse can be eliminated using the
circuit shown in Fig. 12."* That figure's topology — `R` from `VCC` to `nR̄D`, `C` from
`nR̄D` to `GND` — replaces what used to be a hard `+3.3V` tie on pin 3, so a spurious edge on
`1A`/`1B` right at power-up (rail still settling) can't produce a spurious `1Q` pulse, which
for this unit would mean a momentary unwanted PTT key at boot.

**Populated** (confirmed against the live schematic):
- **`R67`** = 10kΩ, `+3.3V` → pin 3 — matches the resistor value already standardized
  elsewhere on this sheet (`R44`/`R45`/`R59`/`R61`/`R64`/`R65`/`R66`)
- **`C83`** = 0.1µF/25V, pin 3 → `GND`
- τ = R×C = 10kΩ × 0.1µF = **1ms**; `1R̄D` crosses the input threshold roughly **0.7–2.3ms**
  after power-up (0.69τ at a 50%·V_CC threshold, up to ~2.3τ at a conservative 90%·V_CC
  threshold) — comfortably longer than a typical LDO's turn-on transient, and negligible
  next to the 10s main TX-timeout pulse (§ 3), so it doesn't delay real operation.

**Fast-discharge diode — `D13` (`BAS116`, LCSC `C232527`, onsemi, SOT-23)**, in parallel with
`R67`: cathode → `+3.3V`, anode → pin 3/`C83` node (same reference design as this board's
`D2`/`D3`/`D5`, reused rather than adding a new diode part). Without it, `C83` can only bleed
down through `R67` back into a *collapsing* `+3.3V` rail — no faster than the rail itself
decays, and not at all if power drops only briefly. On a fast power-cycle, `C83` could still
be partially charged when `+3.3V` returns, giving a shorter (or missing) reset pulse on the
very next power-up — the scenario this circuit exists to handle. `D13` fixes that: reverse
biased (inert, zero loading) during normal charge-up, forward biased the instant pin 3's
node voltage exceeds the decaying rail, dumping `C83`'s charge back into `+3.3V` and letting
`C83` track the rail down to ~0V every time. Same technique the datasheet itself uses one
section later (§11.3, Fig. 13, `D_EXT` across `R_EXT`) for the `CEXT`/`REXT` timing network,
though that one's sized for large surge current (a 47µF `CEXT`); `BAS116` is plenty for this
much smaller 0.1µF node.

**Still open**: unit B (`2R̄D`, pin 11) doesn't have this treatment — it's hard-tied to `GND`
instead (see pin table above), a safe placeholder since unit B isn't designed yet, not a
power-up guard.

## 4. Open design questions (nothing here is schematic yet)

- **`POWER_PB_SIGNAL` can't feed unit B directly, despite how `open_items.md` phrases it.**
  `POWER_PB_SIGNAL` is pulled up to `VRAW` through R45 (`controller_platform.md`), so it
  swings across `VRAW`'s full range (up to ~14.4V) — not a 3.3V/5V logic level. The
  74HC123's own absolute-max rating is `VI` ≤ `VCC` + 0.5V with `VCC` capped at 7V (Table 4,
  limiting values). Raw `POWER_PB_SIGNAL` already couldn't go straight into `GPIO38` for
  the same reason — that's exactly why `Q9` (BSS138) exists as a level-shift stage there
  (`esp32s3_pinout.md`, pin 31). Unit B's trigger will need the same kind of level-shift
  stage in front of it, not a direct wire.
- **VCC rail choice — 3.3V vs 5V — not yet decided.** Both are within the datasheet's
  2.0–6.0V recommended range. 3.3V would match the MCU GPIO logic that both trigger paths
  ultimately originate from (unit A's retrigger pulse, and a level-shifted `POWER_PB_SIGNAL`
  for unit B via a `Q9`-style stage); 5V would match the SA818S/`VBAT` domain the PTT
  circuit downstream of unit A ultimately touches. Pick one before laying out the timing
  RC networks, since it also shifts `K` slightly (§ 3).
- **Stale cross-reference:** `audio_ptt_path.md` § 3 still describes a "MCU PTT-intent GPIO
  (`IO15`) → 74HC123 TX-safety gating" chain. `esp32s3_pinout.md`'s own live-schematic audit
  (item 7 in its notes) says that two-GPIO AND-gate scheme isn't what got built — PTT today
  is a single GPIO (`IO16`) straight into `Q10`, and `IO15` is free again. Worth updating
  `audio_ptt_path.md` § 3 once the real 74HC123 gating point is designed, so it stops citing
  a scheme that was already abandoned.
- **Exact PTT-gating insertion point** (unit A) — series with `Q10`'s gate, its drain, or
  ahead of the still-unselected PTT-keying optocoupler (`ic_inventory.md`) — undecided.
- **Exact force-off override mechanism** (unit B) — how the output edge actually forces
  `PWR_LATCH` low against a possibly-still-asserting `Q6` — undecided; flagged in
  `open_items.md` as "next thing being worked on," not yet started.
- **Power-up glitch handling — resolved for unit A, still open for unit B.** Unit A now has
  the datasheet's Fig. 12 RC network plus a fast-discharge diode on `1R̄D` (pin 3) — see
  § 3.1 (`R67`/`C83`/`D13`). Unit B's `2R̄D` (pin 11) is still just hard-tied to `GND`
  (permanent reset, a safe placeholder) — it'll need the same treatment once unit B's
  design (force-off override mechanism, trigger source) is actually decided, to avoid a
  spurious force-off assertion at boot once it's live.

## 5. Reference documents

- [`ic_inventory.md`](ic_inventory.md) — Nexperia `74HC123D,653` sourcing (LCSC `C5597`,
  SOIC-16/SOT109-1, $0.33 5pc).
- [`controller_platform.md`](controller_platform.md) § TX safety, § Power sequencing — the
  two design intents this part serves.
- [`open_items.md`](open_items.md) — the force-off item that names this part as the leading
  option.
- [`audio_ptt_path.md`](audio_ptt_path.md) § 3 — the PTT chain this gates into (see the
  staleness flag in § 4 above).
- [`esp32s3_pinout.md`](esp32s3_pinout.md) — `IO16`/`IO15` (PTT-intent), `IO38`/`IO48`
  (hold-to-power-off / self-latch) pin assignments.
- **[Nexperia 74HC123;74HCT123 datasheet](https://assets.nexperia.com/documents/data-sheet/74HC_HCT123.pdf)**,
  Rev. 13 (2024-02-21) — pin table (§5), function table (§6), timing/retrigger formulas
  (§10, footnotes [2]/[3]), application info and power-up circuit (§11, Fig. 11/12).
