# Battery/USB Power Mux (TPS2121) — Implemented

**Status: implemented.** All three TPS2121s are placed and wired in `power.kicad_sch`:
`U3` (Stage 1, VRAW combining), `U2` (Stage 2, 5V rail), `U1` (Stage 3, 3.3V rail). The
`ST`-combiner AND gate is also placed: `U4` (74LVC1G08). See
[power_input_combining.md](power_input_combining.md) for Stage 1's own write-up (that doc
already reflects the current as-built schematic).

## Why replace the LM74610 pair

The board now has three possible power sources instead of two: the 18650 pack, the
12–14.5V Powerpole input, and USB VBUS (via the CH334R hub's upstream port — see
[controller_platform.md](controller_platform.md)). The requirement driving this change:

- Battery (18650 or Powerpole, whichever is present) **always powers the system when present**,
  at full SA818 TX power.
- If no battery is present, **USB alone can power the entire board**, but the SA818 TX power
  must be software-limited to stay within USB's current budget.
- Only one source should ever drive a rail at a time — no backfeeding a USB host or the battery
  charge path.
- Firmware needs a cheap way to know which mode it's in.

TI's **TPS2121** (2.8–22V priority power MUX, integrated FETs, seamless switchover, built-in
reverse-current blocking) covers this in one part family across every voltage domain in the
design (5V USB, 7.4–14.4V battery/Powerpole) — see the [ideal-diode-OR vs. TPS2121
tradeoff discussion, session 2026-09-19] for the fuller comparison against LTC4416/LTC4413/
MAX17614.

LCSC: **C485916** (`TPS2121RUXR`) / **C2156025** (`TPS2121RUXT`), VQFN-HR-12 (2×2.5mm), ~$1.2–1.5.

## Architecture — three TPS2121s

```
                         ┌─ IN1 (priority) ── 18650/Powerpole merge
Stage 1 (pre-regulation) │                     [replaces the 2× LM74610 pair]
  VCOMP mode,            └─ IN2 ────────────── (n/a — see note below)
  highest-voltage-wins           │
                                 OUT ── PWR_IN_SELECT ──┬─[Buck+LDO]── BATT_5V_REG
                                                          └─[Buck+LDO]── BATT_3.3V_REG

Stage 2 (post-regulation, 5V rail)          Stage 3 (post-regulation, 3.3V rail)
  IN1 (priority) ── BATT_5V_REG                IN1 (priority) ── BATT_3.3V_REG
  IN2 ────────────── USB_5V (raw VBUS,          IN2 ────────────── USB_3.3V (LM1117-3.3
                      via CH334R hub)                              on the USB-C sheet)
  OUT ── SYSTEM_5V                              OUT ── SYSTEM_3.3V
  ST ── (see detect combiner below)             ST ── (see detect combiner below)
```

**Note on Stage 1:** the 18650 pack and Powerpole input are still combined with a single
2-input VCOMP-mode TPS2121, same role the two LM74610s + AO3400As currently fill — highest
present voltage wins passively, no priority pin biasing needed since Powerpole (12–14V) is
inherently higher than the battery pack (7.4V nominal) whenever both are present.

**Stages 2 and 3 are set to priority = battery** (`PR1` biased so `BATT_x_REG` is IN1), *not*
USB — this was reconsidered mid-design. Setting priority to USB would mean a healthy, present
battery gets silently overridden the instant a USB cable is plugged in (e.g. for firmware
flashing), dropping TX power for no reason. Priority = battery preserves "battery present →
full power" unconditionally, and still yields a free USB-detect signal via `ST` (see below) —
there was no need to trade one for the other.

## `ST` pin behavior (per TPS2121 datasheet, Table 9-3)

- `ST` = **high** → IN1 (battery-derived rail) is powering the output, or output is Hi-Z.
- `ST` = **low** → IN2 (USB-derived rail) is powering the output.

With priority = battery on both Stage 2 and Stage 3, `ST_5V` and `ST_3V3` should always agree
in normal operation — both high (battery, full power) or both low (USB-only, throttled).
They're two independent chips, though, so treat disagreement as "USB-only" (fail-safe toward
the lower-power state) rather than assume they're always in lockstep.

## Single-GPIO USB-detect combiner

Only one spare ESP32-S3 GPIO is available for this (see
[esp32s3_pinout.md](esp32s3_pinout.md) for the current pin budget), but there are two `ST`
signals to monitor. Since we want the GPIO to read **"full power" only when both rails agree
they're on battery**, and **"throttle" if either rail has fallen back to USB**, the combining
function is a 2-input AND (active-high = battery/full power):

```
DETECT_GPIO = ST_5V AND ST_3V3
```

Both `ST` outputs are push-pull CMOS (not open-drain), so they can't just be wired together —
an actual gate is needed. Two options were considered:

1. Diode-AND (passive, no IC) — a pull-up resistor with one small-signal diode from each `ST`
   pin into the node. Cheapest option, but not what was built.
2. **74LVC1G08 (single 2-input AND gate, SOT-23-5)** — cleaner logic levels. **This is the
   one built**: placed as `U4` in `power.kicad_sch`.

## Open items

- [ ] Resistor divider values for Stage 1's VCOMP comparator (if any biasing is needed beyond
      the default) and Stages 2/3's `PR1` battery-priority biasing — not yet calculated.
- [ ] Confirm actual available GPIO for `DETECT_GPIO` against the current ESP32-S3 pin budget
      (per `esp32s3_pinout.md`, no GPIOs are currently spare — this will need to reclaim one).
- [ ] `ILM` current-limit resistor sizing for all three TPS2121s once real rail currents
      (SA818S TX current draw at full vs. throttled power) are known.
- [ ] Update the flow diagram in [power_architecture.md](power_architecture.md) to show
      three sources and the mux stages, if it doesn't already.
