# 74HC123 — Dual Retriggerable Monostable — Pinout & Planned Interconnects

Covers the dual retriggerable monostable multivibrator (Nexperia **74HC123D,653**, LCSC
`C5597`, SOIC-16/SOT109-1 — see `ic_inventory.md`) with **two independent planned uses on
this board**: one section as the TX-safety PTT hardware timeout (`controller_platform.md`
§ TX safety), the other — the "spare half" — as the hardware force-off failsafe
(`open_items.md`). KiCad symbol `74xx:74HC123` / footprint
`Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` (stock libraries, both pin-verified against
Nexperia's own datasheet — no library fix needed).

**This part is not placed in the schematic yet** (`open_items.md`: "the 74HC123 isn't
placed in the schematic either way"). Everything below is a planning document, same rigor
level as `vhf_matching_detector.md` before that ladder existed: real pin functions from the
datasheet, but "planned tie/connection" is intent pulled from `controller_platform.md` /
`open_items.md` / `audio_ptt_path.md`, not something to treat as already wired. Two of the
four trigger nets involved (the retrigger source for the TX-timeout half, and the level of
`POWER_PB_SIGNAL` feeding the force-off half) depend on design decisions that aren't made
yet either — flagged individually below, and again in § 4.

## 1. Full pin table

Reference: Nexperia `74HC_HCT123` datasheet Rev. 13 (2024-02-21), Table 2 (pin
description) and Table 3 (function table), SO16/SOT109-1 pinout.

| Pin | Name | I/O | Function (per Nexperia's own datasheet) | Planned tie/connection |
|---|---|---|---|---|
| 1 | 1A | I | Negative-edge triggered input, unit A (Schmitt) | **Unit A = TX-safety timeout.** Not yet decided whether A or B is the active retrigger input for this half — see § 3. If unused, tie to the level the function table calls for with the other input active (function table: triggering needs `nRD`=H; with `nB`=H, a falling edge on `nA` triggers — so if B is the active input instead, tie `1A` low). |
| 2 | 1B | I | Positive-edge triggered input, unit A (Schmitt) | Mutual with `1A` above — whichever of A/B carries the periodic MCU "keep transmitting" pulse from the PTT-intent path; the other ties to its inactive level. |
| 3 | 1R̄D | I | Direct reset LOW (async, overrides everything) + positive-edge triggered input | Planned: pulled to VCC (inactive) unless a system-level "kill PTT now" reset source is defined. Also see the datasheet's own power-up-glitch note (§ Application information, Fig. 12) — an R̄D-side RC network held low briefly after power-up avoids a spurious startup pulse, which here would mean a spurious momentary PTT key. Not yet designed. |
| 4 | 1Q̄ | O | Active-LOW output, unit A | Planned, with `13`/`1Q`: series-gates the existing PTT path (`IO16` → `Q10` → `J11` → SA818S/HT `PTT`, see `sa818_pinout.md`/`audio_ptt_path.md`) per `controller_platform.md` § TX safety — "gates the PTT line in series with the MCU's own PTT-intent GPIO." Exact insertion point (ahead of `Q10`'s gate, in series with its drain, or ahead of the still-unselected PTT-keying optocoupler) is undecided — open item. |
| 13 | 1Q | O | Active-HIGH output, unit A | See `1Q̄` above — whichever polarity fits the eventual gate implementation. |
| 14 | 1CEXT | — | External timing capacitor connection, unit A | Planned RC timing network for the **2-minute hard TX ceiling** — see § 3 for calculated starting values. Per the datasheet's own app-note (Fig. 11): cap goes between this pin and `15`/`1REXT/CEXT`. |
| 15 | 1REXT/CEXT | — | External timing resistor+capacitor connection, unit A | Resistor to VCC, per the same app-note figure. See § 3 for values. |
| 5 | 2Q | O | Active-HIGH output, unit B | **Unit B = spare-half hardware force-off failsafe** (`open_items.md`). Planned to assert an override of `PWR_LATCH`/`Q6`'s hold (`controller_platform.md` § Power sequencing) if the software hold-to-power-off timeout is exceeded. The actual override mechanism — how a one-shot output edge forces `PWR_LATCH` low against a possibly-still-asserting `Q6` — is the main undecided piece here, not just an RC value. |
| 6 | 2CEXT | — | External timing capacitor connection, unit B | RC target: "longer than the software hold threshold" (`open_items.md`). That firmware threshold isn't documented yet (hold-to-power-off firmware is itself still TBD per the same doc), so no concrete value can be calculated for this half yet — blocked on a firmware constant, unlike unit A's timing. |
| 7 | 2REXT/CEXT | — | External timing resistor+capacitor connection, unit B | Same block as `2CEXT` above. |
| 8 | GND | — | Ground | Planned: `GND`. |
| 9 | 2A | I | Negative-edge triggered input, unit B (Schmitt) | Planned force-off trigger, per `open_items.md`'s leading option: "triggered directly by SW3 (or its isolated `POWER_PB_SIGNAL` node)." **See the voltage-domain flag in § 4 — this can't actually be direct.** |
| 10 | 2B | I | Positive-edge triggered input, unit B (Schmitt) | Mutual with `2A` — same flag applies. |
| 11 | 2R̄D | I | Direct reset LOW (async) + positive-edge triggered input, unit B | Same treatment as `1R̄D` (tie inactive, consider the Fig. 12 power-up RC). Also the natural place to let firmware abort the hardware failsafe once a normal software power-off succeeds in time, if that's ever wanted — not yet decided. |
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

## 3. Unit A — TX-safety timeout: starting RC values

Target: **2-minute (120s) hard ceiling** per `controller_platform.md` § TX safety.

Datasheet's own pulse-width formula (§10, footnote [2], valid for `CEXT` > 10nF):

```
tW (ns) = K × REXT (kΩ) × CEXT (pF)
K ≈ 0.45 at VCC = 5.0V, ≈ 0.55 at VCC = 2.0V (Fig. 6 — K drifts with VCC, "typical" only)
```

Recommended `REXT` range per the datasheet's own dynamic-characteristics table: 2kΩ–1000kΩ
at VCC = 5.0V. Solving for `tW` = 120s = 1.2×10¹¹ ns at K = 0.45:

| REXT | Required CEXT | Notes |
|---|---|---|
| 470kΩ | ≈ 560µF | Mid-range R, needs a real electrolytic/tantalum at this value — not a ceramic. |
| 1MΩ (datasheet's own max) | ≈ 270µF | Smaller, cheaper cap; R right at the recommended ceiling. |

**Rigor note, same as this board's other tuned RC/LC values (matching network, mic
attenuator):** this is a calculated starting point from the datasheet's typical-K formula,
not a bench-verified number — `K` itself is only characterized at the 2V/6V endpoints in
the datasheet (Fig. 6), and electrolytic/tantalum tolerance at these values is loose.
Populate one of the pairs above, then verify the actual timeout against a bench timer
before trusting it for the real 2-minute ceiling. Per the datasheet's own app-note (Fig.
11): `CEXT` connects between pins 14/6 and 15/7, `REXT` from pin 15/7 to VCC.

Also relevant to the retrigger-pulse firmware: the datasheet gives a **minimum retrigger
interval** formula (§10, footnote [3]) — `t_rtrig ≈ 30 + 0.19×REXT×CEXT^0.9 +
13×REXT^1.05` ns (REXT in kΩ, CEXT in pF) — the MCU's periodic "still transmitting" pulses
need to be spaced at least this far apart to reliably retrigger rather than being missed;
at the values above this works out to a floor far below any sane firmware polling
interval, but it's a real constraint worth having in the design record.

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
- **Power-up glitch handling** (both halves) — the datasheet's own Fig. 12 circuit (an RC
  network holding `nR̄D` low briefly after power-up) prevents a spurious pulse at boot; for
  unit A that would mean a spurious momentary PTT key, for unit B a spurious force-off
  assertion. Not yet designed either.

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
