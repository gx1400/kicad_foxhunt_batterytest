# GPS — u-blox MAX-M10S ↔ ESP32-S3 Interconnect Plan

Not yet in the schematic — this is the interconnect plan for the next placement pass,
covering the GPS/GNSS module (u-blox MAX-M10S-00B, already chosen — see `ic_inventory.md`),
its link to the ESP32-S3, its backup-power domain (CR2032 + supercap, diode-ORed), and the
RF input to a vertical SMD SMA connector. See `controller_platform.md` § GPS for the
existing high-level decisions this plan builds on (UART+I2C+PPS module, external active
antenna via SMA, backup domain kept separate from the RTC's own coin cell).

## 1. Part & real pin table

**Part: u-blox MAX-M10S-00B** (18-pad LCC module, `C4153167`). Pin table, from u-blox's own
*MAX-M10S Integration Manual* (UBX-20053088 - R05), § 1.3:

| Pin | Name | I/O | Function | Planned tie/connection |
|---|---|---|---|---|
| 1, 10, 12 | GND | – | Ground | `GND` |
| 2 | TXD | O | UART TX (module → host) | ESP32-S3 IO18 (`U1RXD`, per `esp32s3_pinout.md`) |
| 3 | RXD | I | UART RX (host → module) | ESP32-S3 IO17 (`U1TXD`) |
| 4 | TIMEPULSE | O | Time pulse (PPS) | ESP32-S3 IO5 (interrupt-capable, per `esp32s3_pinout.md`) |
| 5 | EXTINT | I | External interrupt (wake source) | Not used this design — leave open |
| 6 | V_BCKP | I | Backup domain supply | Backup power network (§ 3) |
| 7 | V_IO | I | Digital IO supply | `+3.3V` |
| 8 | VCC | I | Main supply (core + RF) | `+3.3V` |
| 9 | RESET_N | I | Reset, active-low, internal pull-up to V_IO | Leave open (datasheet default — no external cap, would trigger spurious resets) |
| 11 | RF_IN | I | GNSS signal input, 50Ω, internally DC-blocked | Direct 50Ω trace → SMA connector, with antenna bias-tee (§ 4) |
| 13 | LNA_EN | O | On/off external LNA or active antenna | Not used this design — see § 4 note |
| 14 | VCC_RF | O | Filtered RF-domain supply (derived from VCC via internal ferrite bead) | Antenna bias-tee supply (§ 4) |
| 15 | VIO_SEL | I | V_IO voltage select | **Leave open** — selects 3.3V mode (grounding selects 1.8V, which this board doesn't use) |
| 16 | SDA | I/O | I2C data | Not used this design — leave open (UART-only, per `esp32s3_pinout.md` judgment call #1) |
| 17 | SCL | I | I2C clock | Not used this design — leave open |
| 18 | SAFEBOOT_N | I | Safeboot mode select, internally tied to TIMEPULSE via a 1kΩ series resistor | Leave open for normal operation (default) |

This is **Option 1** from the integration manual's supply design table (§ 4.1.4): VCC and
V_IO both at 3.3V, tied together, `VIO_SEL` left open. Standard 0.1µF local decoupling at
VCC/V_IO, matching this board's convention elsewhere.

## 2. UART + PPS ↔ ESP32-S3

Already reserved in `esp32s3_pinout.md` (UART1, matching the datasheet's own U1TXD/U1RXD
labels): IO17 → module RXD, IO18 → module TXD, IO5 → TIMEPULSE (PPS). No I2C connection —
this design deliberately stays UART-only (simpler, and the standard interface for
NMEA/UBX streams), so SDA/SCL are left open per the pin table above.

## 3. Backup power domain — CR2032 + supercap, diode-ORed

**Separate from the RTC's own coin cell** (per `controller_platform.md`) — this needs its
own holder + supercap footprint, not a tap off `BT3`.

**Why both:** the datasheet's own hardware-backup-mode current spec (electrical
characteristics, *MAX-M10S Data sheet* UBX-20035208) is **I_V_BCKP ≈ 32µA typical** at
V_BCKP=3.3V — two orders of magnitude higher than the RTC's own ~3pA-leakage-diode-class
backup draw, because this domain is holding GNSS orbit/ephemeris data (BBR), not just a
clock register. That's still small in absolute terms, but worth sizing deliberately rather
than assuming "anything works":

- **CR2032** (typical ~220mAh): at a continuous 32µA draw, that's a theoretical
  ~285 days of hardware-backup-mode current alone (real shelf life will be dominated by the
  cell's own self-discharge over long idle periods, same caveat as the RTC's coin cell) —
  comfortably covers the days-to-weeks gap between fox-hunt events that ephemeris data
  (~4 hours validity per the datasheet) doesn't even need to survive; its real job here is
  keeping the RTC/BBR alive across *storage* gaps, same role as the main RTC's own cell.
- **Supercap sizing**: target bridging 30–60 minutes at 32µA without touching the coin
  cell's capacity — `Q = I×t` ≈ 0.0576–0.115 C; using the ~1.3V usable swing between a
  topped-up ~3.0V (after diode drop) and the datasheet's V_BCKP minimum (1.65V), that's
  **C ≈ Q/ΔV ≈ 0.04–0.09F**.
- **Supercap chosen: Eaton KR-5R5V224-R** — 0.22F, 5.5V, coin-cell through-hole (5mm lead
  spacing, PC pins), ESR 75Ω, 11.5mm dia × 12.7mm height. **DNP / not LCSC-sourced** — using
  a part already on hand rather than ordering through LCSC. 0.22F is ~2.5–5× the calculated
  0.04–0.09F target, which just means a longer bridge time for free (no downside).
  Voltage rating (5.5V) is well above anything this part will actually see (see topology
  below) — no risk of exceeding it.
- **Backup network topology — final, settled design** (superseding an earlier version of
  this doc that both mis-stated the diode arrangement and assumed a 5V charge source before
  the overvoltage risk was caught):
  - `+3.3V` → **diode A** (anode on `+3.3V`, cathode on the shared node) — same role as the
    RTC's `D2`.
  - `CR2032` (`BT4`) → **diode B** (anode on `BT4`, cathode on the shared node) — same role
    as the RTC's `D3`.
  - `+3.3V` → **diode C** → **R (1kΩ)** → **supercap** — a dedicated charge branch, *not*
    off the 5V rail. Charging from 3.3V (not 5V) is the key safety decision: `V_BCKP`'s
    absolute max is **3.6V** with **zero margin** above its own normal operating max (same
    value, per the real datasheet — "product is not protected against overvoltage... voltage
    spikes must be limited"), so charging from 5V and relying on a diode's forward-voltage
    drop to claw the excess back off is unreliable — Vf is current-dependent, not a fixed
    clamp, and at this circuit's ~32µA backup current a diode's Vf is far too small (and too
    variable) to guarantee staying under 3.6V. Charging from 3.3V removes the problem
    entirely: the cap settles at ~3.3V minus a diode drop, safely clear of 3.6V regardless of
    the exact Vf.
  - **Supercap → diode D → shared node** — the discharge-out path, one-way (blocks the node
    from charging the cap through this branch, keeping the charge and discharge paths
    independent, matching the isolation already used for the coin cell).
  - `V_BCKP` (U18 pin 6) ties directly to the shared node — no additional diode or resistor
    between the node and the pin itself (only diode C's resistor is in the cap's own charge
    branch, off to the side, per the integration manual's "avoid high resistance on the
    V_BCKP line" warning).
  - **All four diodes: Nexperia BAS116LT1G** (reuse the same part already verified and used
    for the RTC's `D2`/`D3`) — **not** a Schottky or fast-recovery part, despite its higher
    forward drop (~0.4–0.5V at this circuit's actual 32µA, per the real datasheet's Figure 2
    curve — not the 0.9V-at-1mA table spec, and not as low as a rough sub-mA extrapolation
    either). A Schottky's lower Vf comes from a lower junction barrier, which also means
    meaningfully higher reverse leakage (often nA–µA class vs. this part's verified ~3pA) —
    exactly the tradeoff this project already rejected once for the RTC's own backup diodes.
    Trading leakage for a few hundred mV of headroom this circuit doesn't need would be a
    bad trade for a domain whose whole point is lasting months on a coin cell.
- **Coin cell holder**: reuse the same part already placed for the RTC (`BS-08-B2AA020-R`,
  kicad_gx_library) rather than sourcing a second holder family, unless a different
  footprint is wanted for layout reasons. Already placed as `BT4`, currently wired directly
  to `V_BCKP` with no diode yet — matches the "not wired in yet" state confirmed this
  review; needs diode B inserted between them per the topology above.

## 4. RF input — SMA connector + active antenna bias-tee

**Vertical SMD SMA connector** direct to `RF_IN` via a 50Ω trace — MAX-M10S integrates its
own LNA + SAW filter internally (integration manual § 4.3), so no external matching network
is needed between the connector and the pin; keep the trace short, per the datasheet's own
layout guidance (§ 4.4).

**Connector footprint — through-hole for now, deliberately.** `J10` is currently placed
with `Connector_Coaxial:SMA_Amphenol_132134_Vertical`, a through-hole part (confirmed via
its pad data: 5 thru-hole pads) — chosen on purpose, using a part already on hand for
bring-up, not a placement mistake. **Follow-up planned** (tracked in `open_items.md`): swap
to a true vertical SMD SMA before final layout. Candidate researched: **MyAntenna
A-SMA-KE-16.5A (`C22467617`)** — SMD, "Board Side" (vertical), 2,916 in stock, $0.43–0.83 —
paired with KiCad's `Connector_Coaxial:SMA_Wurth_60312102114405_Vertical` footprint
(confirmed genuinely SMD: all 3 pads `pad_type: smd`, no drills — the only truly SMD
vertical SMA footprint in KiCad's stock library). **Not yet cross-checked**: whether this
footprint's exact land-pattern dimensions match the MyAntenna part's real datasheet drawing
(LCSC's automated PDF download kept failing) — verify before committing, same rigor as this
project's other footprint checks.

**External active antenna, per `controller_platform.md`'s existing decision** — needs a
bias-tee network to feed antenna power onto the same RF line, per the integration manual's
own reference circuit (§ 4.3.4, Figure 28) and its appendix component table:

| Ref | Function | Value (per u-blox's own reference design) |
|---|---|---|
| R51 | Short-circuit current limiter, in series with `VCC_RF` | 10Ω, 5%, 0.25W |
| L3 | Bias-tee feed inductor, `VCC_RF`(via R51) → antenna/RF-line node | 27nH, 5% — high impedance at GNSS L1 (1.575GHz) so RF energy doesn't leak back into the 3.3V-derived supply; needs ≥300mA current rating (Murata LQG15H/LQW15A or Johanson L-07W series, per u-blox's own recommendation) |
| C67 | Series DC-block, antenna/RF-line node → `RF_IN` | 10nF, 10%, 16V, X7R — negligible reactance at L1 (~0.01Ω), so it's electrically transparent to the RF signal while blocking the DC bias from ever reaching `RF_IN` |

**Correction from an earlier version of this doc**: C67 (labeled C14 in u-blox's reference)
is a **series** DC-blocking cap on the main antenna-to-`RF_IN` trace, not a shunt filter cap
from `VCC_RF` to GND as originally described here — verified by tracing the actual placed
wiring (`gps.kicad_sch`, confirmed 2026-09-20) and cross-checking against the physics (a
267Ω reactance from a 27nH inductor placed directly in a 50Ω RF trunk would badly mismatch
it, confirming L3 must sit in the DC-feed branch, not the RF signal path). As built: antenna
(`J10`) and `L3`'s DC-feed node are the same node; `C67` sits between that node and `RF_IN`.
**Verified correctly wired** as of this review.

Fed from `VCC_RF` (pin 14, filtered RF-domain supply, always live whenever `VCC` is
present) directly — **not gated by `LNA_EN`** in this design. `LNA_EN` exists specifically
to power-cycle an external LNA/active-antenna supply during backup/power-save modes to save
current, which this design doesn't need for its first pass (antenna stays powered whenever
the board itself is powered); leave `LNA_EN` unconnected. Revisit only if antenna standby
current becomes a real budget concern later — would mean adding a FET switch gated by
`LNA_EN` in series with this network, not a bigger redesign.

**Not built this pass**: the antenna supervisor (open/short-circuit detection, `ANT_DETECT`/
`ANT_SHORT_N`/`ANT_OFF_N`) — it's a real, documented feature (integration manual § 3.3.1),
but needs 2–3 extra GPIOs reassigned from other functions, and this board is already down to
one genuinely spare pin (IO3, per `esp32s3_pinout.md`). Skipped for the same reason the RF
power-sensing section already accepts forward-power-only (no reflected/SWR) — diminishing
return for the GPIO budget it costs.

## 5. Reference documents

- **[MAX-M10S Integration Manual](https://cdn.sparkfun.com/assets/5/c/a/0/b/MAX-M10S_IntegrationManual_UBX-20053088.pdf)**
  (UBX-20053088-R05) — source for the pin table, backup-mode description, supply design
  options, and antenna bias-tee reference circuit/component values in this doc.
- **[MAX-M10S Data sheet](https://cdn.sparkfun.com/assets/7/5/9/a/a/MAX-M10S_DataSheet_UBX-20035208.pdf)**
  (UBX-20035208) — source for the V_BCKP electrical characteristics (32µA typical hardware
  backup current) used in § 3's sizing math.
- **[u-blox M10 antenna list](https://cdn.sparkfun.com/assets/5/c/a/0/b/MAX-M10S_IntegrationManual_UBX-20053088.pdf)**
  (Integration Manual Appendix C.1) — example passive/active L1 GNSS antennas (Taoglas,
  Amotech, INPAQ) if a specific model needs picking for the external SMA antenna.

## Open items

- **V_BCKP backup network not wired yet** — `BT4` (CR2032) is placed but tied directly to
  `V_BCKP` with no diode; still need diode A (`+3.3V`→node), diode B (`BT4`→node), diode C
  (`+3.3V`→R→supercap charge branch), diode D (supercap→node), the supercap (chosen: Eaton
  `KR-5R5V224-R`, 0.22F/5.5V through-hole, DNP/not LCSC), and the ~1kΩ charge-limiting
  resistor. All four diodes: `BAS116LT1G`. See § 3 for the full topology and reasoning.
- **`J10`'s footprint needs swapping** from the through-hole `SMA_Amphenol_132134_Vertical`
  to the SMD `SMA_Wurth_60312102114405_Vertical` (or whatever footprint matches the final
  chosen SMD SMA part) — land-pattern match not yet cross-checked against a real datasheet.
- Antenna supervisor (open/short detection) explicitly not built this pass — revisit only if
  a GPIO frees up (none currently spare — see `esp32s3_pinout.md`) and antenna-fault
  detection becomes worth the cost.
- No specific antenna model chosen yet — external, user-supplied via SMA; Appendix C.1 above
  lists u-blox's own suggested examples if one needs picking.
- `EXTINT`, `RESET_N`, `LNA_EN`, `VIO_SEL`, `SDA`, `SCL`, `SAFEBOOT_N` — all correctly left
  open with `no_connect` markers (7 total), matching the plan. `VCC_RF` and `RF_IN` are
  correctly wired through the bias-tee, not left open.
