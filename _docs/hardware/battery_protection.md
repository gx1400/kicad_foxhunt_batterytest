# Battery Protection & Fuel Gauge — MAX17320

- 2S1P 18650 configuration. CELL2/CELL3 pins shorted to CELL1 (not BATTS) — matches the "2×RBAL" balancing path used for the top cell.
- **ModelGauge m5 EZ** fuel gauge mode — no per-cell characterization required.
- **Sense resistor:** 5mΩ, 4-terminal Kelvin (2512 package). Must be explicitly configured in nonvolatile memory during the config wizard — the IC defaults to 2.5mΩ (RSenseSel code 1), which would silently halve every current/capacity reading if left unset.
- **Cell balancing:** RBAL1 (CELL1) = 150Ω, RBAL4 (BATTS) = 150Ω. Bottom cell ≈ 26mA, top cell ≈ 13.6mA (routes through both RBAL1 and RBAL4 in series).
- **Zero-volt charge recovery:** RZVC = 820Ω, sized for ~10mA recovery current at an assumed 8.4V charger CV rail.
- **Permanent-failure fuse:** 3-terminal fuse (Littelfuse ITV4030L1212 or equivalent), heater triggered by PFAIL through an AO3400A N-FET.
- **CHG/DIS FETs:** back-to-back dual N-FET pair, gates on CHG/DIS, sources split — one referencing raw battery (IN side), one referencing the post-FET system node (PCKP side).
- **RIN:** 10Ω per datasheet spec, in series with IN.
- **PFAIL, unused comms/thermistor pins:** ALRT, SCL/OD, SDA/DQ, TH2–TH4 left as no-connects; only TH1 populated.

## Ground domain split (GND vs. GNDREF)

Two separate ground nets, bridged **only** by the current-sense resistor (R3):

- **GNDREF** (battery-referenced island): U1's GND pin, CSP, battery negative terminal, IN/CP/REG2/REG3-area bypass caps.
- **GND** (system-referenced): CSN, all downstream regulator grounds, output-side bypass caps.

This split is what makes R3 actually measure current rather than being bypassed by a parallel ground path. Confirmed correct in the current netlist.

*Don't "fix" an apparent GND/GNDREF split if you spot it during review — it's intentional.*
