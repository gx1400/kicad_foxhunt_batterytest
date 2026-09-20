# Dual-Input Power Combining

- **TPS2121RUXR** (U3 in `power.kicad_sch`) combines the two input branches into VRAW,
  operating in VCOMP (voltage-comparator/priority) mode — it picks whichever of its two
  inputs is higher and passes that through, no external FETs or ORing controller needed.
  Superseded an earlier LM74610-Q1 + external-N-FET (AO3400A) ideal-diode design; this
  doc previously described that superseded approach and was out of date.
- IN1 = `PWR_18650` (2S1P pack, via the battery-sensing divider network), IN2 =
  `PWR_POWERPOLE` (external 12–14.5V input) — see [power_architecture.md](power_architecture.md)
  for the overall signal flow. Output (`OUT`) is VRAW, feeding the [VRAW soft-latch](controller_platform.md#power-sequencing--discrete-soft-latch)
  and then Power Regulation.
- Same TPS2121 part (in a separate VCOMP-mode instance per rail) is reused downstream for
  USB-vs-battery priority muxing on the regulated 5V/3.3V rails — see
  [regulation.md](regulation.md).
- Extensive solder-jumper isolation points throughout this section and the regulation
  stages, letting each stage — battery branch, 12V branch, buck input, buck-to-LDO
  handoff — be bench-tested independently before trusting the full chain.
