# Dual-Input Power Combining

- **LM74610-Q1** ideal-diode controllers, one per input branch, each driving an external N-FET (AO3400A) as a near-lossless "smart diode."
- Chosen over passive Schottky diode-ORing specifically to minimize voltage drop and power dissipation (roughly 9x lower loss at ~2A vs. a Schottky pair).
- VCAP on each LM74610: 1µF, 16V, X7R.
- Extensive solder-jumper isolation points throughout this section and the regulation stages, letting each stage — battery branch, 12V branch, buck input, buck-to-LDO handoff — be bench-tested independently before trusting the full chain.
