# Audio / PTT Path — I2S DAC ↔ ESP32-S3 ↔ SA818S / External HT

Not yet in the schematic — this is the interconnect plan for the next placement pass,
covering the I2S DAC (PCM5102APWR, already chosen — see `ic_inventory.md`), its link to the
ESP32-S3, and how its output (and the PTT-keying signal) reach both the onboard SA818S
footprint and the external-HT 3.5mm jack. See `controller_platform.md` § Audio / PTT path
for why this is one shared subsystem serving two destinations, not two separate designs.

## 1. DAC ↔ ESP32-S3 (I2S bus)

**Part: TI PCM5102A** (20-pin TSSOP, `C107671`). Real pin table, from TI's own datasheet
(PCM5100A/5101A/5102A family, SLAS859A):

| Pin | Name | Function | Planned tie/connection |
|---|---|---|---|
| 12 | SCK | System clock input | **Tie to GND.** Selects the DAC's internal PLL auto-clock mode — no master clock (MCLK) needed from the MCU. ESP32-S3's I2S peripheral doesn't need to supply one either; this is the standard simplified hookup used by every hobbyist PCM5102 breakout (Adafruit's included — see § 4). |
| 13 | BCK | Bit clock input | ESP32-S3 IO14 (`esp32s3_pinout.md` — currently documented as I2S BCLK) |
| 14 | DIN | Serial audio data input | ESP32-S3 IO47 (documented as I2S DOUT — MCU transmits, DAC receives) |
| 15 | LRCK | Word clock (L/R select) input | ESP32-S3 IO21 (documented as I2S LRCLK/WS) |
| 16 | FMT | Audio format select (Low = I2S, High = left-justified) | **Tie to GND** — I2S format, matches ESP32-S3's I2S peripheral default |
| 10 | DEMP | De-emphasis control (Low = off) | **Tie to GND** — no de-emphasis needed |
| 11 | FLT | Filter select (Low = normal latency, High = low latency) | **Tie to GND** — normal latency; nothing in this design needs low-latency monitoring |
| 17 | XSMT | Soft-mute control (Low = mute, High = un-mute) | **Needs a decision** — simplest is tie to `+3.3V` (always un-muted, matches PCM5102A's own DirectPath output design which needs no external mute circuit for pop-free power-up per TI's datasheet), but see § 3 for why the switching mechanism might want to drive this from a spare GPIO instead |
| 6 | OUTL | Analog output, 2.1V<sub>RMS</sub> line level | → level-matching network → SA818S/HT (§ 2) — SA818S is mono, only one channel is needed (confirmed: its `MIC_IN` and `AF_OUT` are each a single pin, no L/R pair, per its own datasheet) |
| 7 | OUTR | Analog output, 2.1V<sub>RMS</sub> line level | → standard RC output filter only (no attenuator — nothing mic-level on this leg) → spare JST breakout (§ 2), for future use |
| 1, 8, 20 | CPVDD, AVDD, DVDD | Power, 3.3V | `+3.3V` |
| 18 | LDOO | Internal LDO decoupling (or external 1.8V) | Decoupling cap only, per datasheet — not using external 1.8V supply mode |
| 3, 9, 19 | CPGND, AGND, DGND | Grounds | `GND` |
| 2, 4 | CAPP, CAPM | Charge-pump flying cap | Per datasheet's charge-pump cap network |
| 5 | VNEG | Charge-pump negative rail (-3.3V) | Decoupling only |

Recommended output filter, straight from TI's own datasheet (footnote on the dynamic
performance table): **470Ω series resistor + 2.2nF shunt cap** per channel, into a ≥10kΩ
load. This is the standard simple RC low-pass already anticipated in
`controller_platform.md` ("I2S DAC → RC filter").

**ESP32-S3 side**: IO14/IO21/IO47 already reserved in `esp32s3_pinout.md`. No MCLK pin
reserved — confirmed not needed per SCK-tied-low above.

## 2. DAC output ↔ SA818S / external HT (level matching + switching)

**Confirmed: SA818S is mono** — `MIC_IN` (pin 18) and `AF_OUT` (pin 3) are each a single
pin in its datasheet, no L/R pair. Only `OUTL` is needed for the SA818S/HT path. `OUTR`
is otherwise unused, so it gets the standard 470Ω + 2.2nF output filter (same as `OUTL`,
no attenuator — there's no mic-level target on this leg) and is broken out to its own
spare JST connector for future use.

**Now placed in the schematic** (`audio.kicad_sch`, verified 2026-09-20): both channels'
output filters and the charge-pump network are built and match this doc's plan —
- `OUTR` → R47 (470Ω) → C63 (2.2nF to GND) → **J9** (spare 3-pin JST, pins 2/3 to GND) — done.
- `OUTL` → R48 (470Ω) → C64 (2.2nF to GND) → currently ends at an open node, awaiting the
  attenuator network (§ below) and the SA818S/HT connection once placed.
- Charge pump: C55 (2.2µF, `CAPP`↔`CAPM` flying cap) and C56 (2.2µF, `VNEG`↔`GND`
  reservoir cap) — matches TI's Figure 39/41 reference circuit.
- `XSMT` is wired to a live hierarchical label (`I2S_XSMT`) back to the ESP32-S3 sheet
  rather than hard-tied — the tie-off decision below is resolved as GPIO-controlled,
  not tied high; not yet confirmed which physical GPIO drives it.

**OUTL → MIC_IN attenuator — now placed** (`audio.kicad_sch`, verified 2026-09-20):
`R49` (100kΩ, top fixed) → `RV1` (Bourns 3314G, 1kΩ single-turn trimmer,
`Potentiometer_SMD:Potentiometer_Bourns_3314G_Vertical` — real stocked footprint;
note it's single-turn, not 10-turn — no genuine 10-turn part in the LCSC list checked
had a matching KiCad footprint) → `R50` (150Ω floor) → GND. Trimmer's full travel spans
roughly 3–24mV (bracketing the SA818S datasheet's ~10mV reference point), never close to
overdriving the mic input even at either extreme. `R_top` at 100kΩ is comfortably above
the 470Ω filter resistor (won't detune its corner) and the PCM5102A's own ≥10kΩ minimum
load spec.

**DC-blocking cap — placed.** C65 (4.7µF, `Device:C_Small` — non-polarized ceramic, 0603)
sits correctly in series between `RV1`'s wiper and the net that will eventually reach
`MIC_IN`. Sized so the high-pass corner stays under ~70Hz even in a pessimistic
(~500Ω) assumption for the divider's output impedance plus `MIC_IN`'s undocumented input
impedance — comfortably below the ~300Hz low end of a voice/NBFM channel.

**`I2C_MIC_OUT` sheet pin — now added.** The net downstream of C65 has a real sheet pin
on the Audio sheet symbol (root level) exposing it out of the sheet. It currently ends
at a `no_connect` stub on the root sheet (nowhere to route it yet, since SA818S isn't
placed) — expected/fine for now, revisit once SA818S is placed. The `I2C_` prefix on
this net name is still misleading (this is analog audio, not the I2C bus) — rename
whenever convenient (e.g. `AUDIO_MIC_OUT`), no rush since nothing depends on the name yet.

**XSMT — now wired to the MCU, decision made and kept.** `I2S_XSMT` is wired across the
hierarchy to **ESP32-S3 GPIO3** (`I2C_MUTE_XSMT` net) — this was the project's one
remaining genuinely-spare GPIO (`esp32s3_pinout.md` previously flagged it as such); it's
now spent on this. `esp32s3_pinout.md`'s summary line and pin table should be updated to
reflect GPIO3 as used, not spare. Net name carries the same misleading `I2C_` prefix as
above — same low-priority rename recommendation.

**XSMT — placed, but the MCU-override path isn't live yet.** `R46` (10kΩ) pulls
`I2S_XSMT` up to `+3.3V` (matches PCM5102A's DirectPath default-unmuted design). The
hierarchical label exists in `audio.kicad_sch`, but the Audio sheet's parent instance
only exposes 3 sheet pins (`I2S_BCK`/`I2S_DIN`/`I2S_LRCK`) — no `I2S_XSMT` pin yet, so
right now this net is local to the sheet and behaves as a hard pull-up. Add a 4th sheet
pin + GPIO route later if MCU-controlled muting is wanted.

**Critical point, easy to miss:** PCM5102A's output is genuine **line level** (2.1V<sub>RMS</sub>
nominal). SA818S's `MIC_IN` (pin 18) is sized for **microphone-level** signals — its own
datasheet's modulation-frequency test table specifies only **~10mV** typical input for a
1.5/2.5kHz deviation test point. Feeding 2.1V<sub>RMS</sub> straight into `MIC_IN` would grossly
overdrive the modulator (excess deviation, splatter) even though the pin is nominally
labeled "Microphone or line in" — that label describes what it'll pass electrically, not
that it's correctly leveled for a line-level source. **A resistor-divider attenuator
between the DAC output (post-RC-filter) and `MIC_IN` is required**, not optional — exact
divider ratio is a TBD depending on the DAC's actual output swing at whatever digital
gain firmware uses; needs bench-tuning against a real deviation measurement, not just a
datasheet-derived guess.

**Switching between SA818S and the external HT jack — superseded plan, 2026-09-21.**
Earlier framing (jumper vs. PCA9555-controlled switch) is replaced by a simpler mechanical
approach using the external jack itself:

**External jack connector plan.** One standard 3.5mm TRS/TRRS jack (not the K1 module's own
2-jack 3.5mm+2.5mm split — that split exists on the *radio* side; off-the-shelf "K1 to
3.5mm" adapter cables, e.g. the BTECH one sold for Baofeng/Kenwood-style radios, bridge
between a standard PC/phone-style 3.5mm plug and the radio's actual K1 socket, so this
board only needs the standard-headset side). Real K1 pinout for reference (Wildtalk's
documented reference; clone radios vary): radio's own 3.5mm jack is Tip=PTT (short to
ground to transmit), Ring=Mic (radio provides phantom power), Sleeve=5V tap; radio's 2.5mm
jack is Ground/Program/Speaker-out — not replicated here since this design doesn't need the
speaker/program functions.

**Planned wiring**: Tip = attenuated mic-level audio (shared with SA818S `MIC_IN`), Ring =
PTT (shared with SA818S `PTT`), Sleeve = `GND`.

**Switching mechanism: a jack with two independent normally-closed switch contacts**, not
a jumper. Plugging in disconnects the SA818S from *both* the audio feed and the PTT feed
simultaneously (one NC contact per conductor) — important because PTT is otherwise wired
in parallel to both destinations, and without this, keying PTT while something's plugged in
would transmit on both the SA818S and the external radio at once (harmless if the SA818S
has no antenna connected, a real mutual-interference risk if it does). Look for a switched
3.5mm jack part (e.g. Cliff/CUI parts marketed with "SW"/detect contacts) with enough
independent switch poles for this. PTT is a direct short-to-ground per the real K1 spec —
compatible with the already-planned optocoupler/relay-style PTT output (not a raw GPIO
logic level), so the same PTT-keying hardware likely drives both destinations unmodified.

**Open assumption**: routing the SA818S-tuned attenuator output to an arbitrary external
radio assumes similar mic sensitivity — reasonable for most cheap HTs (similar electret-
level inputs) but not verified against a specific target radio.

**RX audio (the other direction)**: SA818S `AF_OUT` (pin 3) — typical 700mV amplitude,
200Ω output impedance per its own datasheet — feeds an external amplifier (SA818S's
`Audio ON` pin 1 auto-controls that amp: outputs LOW to enable it, HIGH to disable —
active-low enable, worth remembering when wiring). This project doesn't currently plan to
route RX audio back into the MCU (no on-board speaker amp chosen yet) — flagging as an
open item if audio monitoring/recording is ever wanted, out of scope for TX-path wiring.

## 3. PTT-keying path

**SA818S `PTT` (pin 5)**: active-low input — "0" forces TX, "1" is RX. Matches this
project's already-documented convention ("PTT line defaults to released... pull resistor
sized accordingly," per `controller_platform.md` § TX safety) — needs a pull-up to keep it
released (RX) whenever the driving GPIO/optocoupler output is undriven or in reset.

**Planned chain**: MCU PTT-intent GPIO (IO15, per `esp32s3_pinout.md`) → 74HC123 TX-safety
gating (not yet built — see `open_items.md`) → PTT-keying optocoupler (still unconfirmed
part — `ic_inventory.md` has 3 scored candidates, PC817-class) → `PTT` pin, isolating the
MCU's 3.3V domain from whatever the SA818S/external-HT PTT circuit actually needs
electrically. Same switching consideration as the audio path applies here too — if the
destination (SA818S vs. external HT) is jumper-selected, the keyed PTT signal needs to
reach whichever one is currently active, ideally through the same switching mechanism
rather than a second independent jumper.

**Two other SA818S pins worth remembering during layout** (not part of the audio/PTT path
itself, but adjacent, real constraints from its own datasheet):
- `PD` (pin 6) — power-down control, "0" = power down, "1" = normal work. Needs a defined
  level (not left floating) — likely tied to whatever gates the module's own power, or a
  simple pull-up if the module should always be enabled whenever its VBAT is present.
- `H/L` (pin 7) — output power select. Datasheet explicitly warns: **"this pin can NOT be
  connected to VDD or high level of CMOS output"** — leave open for high power, or pull to
  a level BELOW logic-high explicitly (not a GPIO driven high) if low power is ever wanted.
  Worth a note directly in the schematic once this is placed, since it's an easy-to-violate
  constraint.

## 4. Reference schematics

Real, open-source references worth reviewing before finalizing values:

- **[Adafruit PCM510x I2S DAC breakout](https://github.com/adafruit/Adafruit-PCM510x-I2S-DAC-PCB)**
  — real, fabricated open-hardware board using this exact chip family (PCM5102/PCM5100).
  Confirms the SCK-tied-low, no-MCLK-needed hookup in practice, not just datasheet theory.
  Full write-up: [Adafruit Learn guide](https://learn.adafruit.com/adafruit-pcm510x-i2s-dac)
  (PDF: [here](https://cdn-learn.adafruit.com/downloads/pdf/adafruit-pcm510x-i2s-dac.pdf)).
- **[NiceRF SA818 datasheet](https://logifind.com/u_file/2108/file/91fb17a776.pdf)** —
  manufacturer's own datasheet, includes a typical application schematic (§6, page 4) that
  is the closest thing to a manufacturer reference circuit for the MIC_IN/PTT/AF_OUT wiring
  — worth comparing against once the attenuator network is sized.
- **[jumbo5566/sa818 on GitHub](https://github.com/jumbo5566/sa818)** — a real open-source
  project ("USB audio CM108 Sound Card PTT Controller + SA818/SR-FRS Programming Tool")
  interfacing a sound-card-level audio output and a PTT control line to this exact module —
  a practical, working precedent for the same audio-level + PTT-isolation problem this
  design is solving, from a different host (PC sound card) instead of an I2S DAC, but the
  MIC_IN attenuation and PTT-isolation concerns are the same.
- **[TI PCM5102A/PCM5100A/PCM5101A datasheet](https://www.ti.com/lit/ds/symlink/pcm5102a.pdf)**
  — source for the pin table and output-filter values in § 1.

## Open items

- Attenuator network (`R49`/`RV1`/`R50`/`C65`, § 2) for `OUTL` → `MIC_IN` — placed in
  `audio.kicad_sch`. Exact trimmer setting still needs a real bench deviation measurement
  once SA818S is placed, not just the calculated nominal target.
- External HT jack: pick a real switched 3.5mm TRS/TRRS part (two independent NC contacts,
  one per audio/PTT conductor — see § 2) and verify it against a real datasheet, same rigor
  as this project's other connector choices. Not yet sourced.
- Confirm which ESP32-S3 GPIO drives `I2S_XSMT` (wired as a live net, not hard-tied —
  tie-off decision is resolved, just needs the specific pin documented here and in
  `esp32s3_pinout.md`).
- `PD` and `H/L` pin handling once SA818S is actually placed (DNP per `ic_inventory.md`,
  but still needs real net connections in the schematic).
- PTT-keying optocoupler part finalization (see `ic_inventory.md` § PTT-Keying Optocoupler).
