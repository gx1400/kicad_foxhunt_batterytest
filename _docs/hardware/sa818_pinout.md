# SA818S-V — Pinout & Interconnect Plan

Covers the onboard VHF transceiver module (G-NiceRF SA818S-V, `U19`, placed as the
`SA818S-V` sheet — `SA818V.kicad_sch`). Real pin table, from NiceRF's own datasheet
(SA818S Rev 1.3 — see § 3). See `audio_ptt_path.md` for the shared DAC/PTT subsystem this
module is one of two destinations for, and `gps_path.md` for the sibling RF module this
board also carries.

## 1. Full real pin table

| Pin | Name | I/O | Function (per NiceRF's own datasheet) | Planned tie/connection |
|---|---|---|---|---|
| 1 | Audio ON | O | Auto-controls an external audio amp — outputs **low** to enable it, **high** to disable. Active whenever the module is powered, regardless of RX/TX state. | Not used this design — no onboard speaker amp. Leave open. |
| 2 | NC | – | Not connected | Leave open |
| 3 | AF_OUT | O | RX audio out (~700mV typ, 200Ω output impedance per earlier research) | Not routed back into the MCU this pass — no onboard speaker amp/audio-monitoring path chosen yet. Leave open; revisit if RX audio monitoring/recording is ever wanted (flagged as an open item in `audio_ptt_path.md`). |
| 4 | NC | – | Not connected | Leave open |
| 5 | PTT | I | Active-low: **"0" forces TX, "1" is RX** | **Wired.** `SA818_PTT_IN` ← shared PTT node (`Q10`/`J11` circuit, see `audio_ptt_path.md`) |
| 6 | PD | I | Power-down control: **"0" = power down, "1" = normal work** | **Not yet wired.** Needs a defined level — floating is not safe (indeterminate power state). Tie high (pull-up to `+3.3V`, or a GPIO if software-controlled power-down is ever wanted) so the module defaults to normal operation. |
| 7 | H/L | I | Output power select. Datasheet's own explicit warning: **"this pin can NOT be connected to VDD or high level of cmos output."** Leave open = high power; pull to a level *below* logic-high (not a GPIO driven high) = low power. | **Not yet wired.** Leave open for high power (simplest, safest choice) unless low-power mode is specifically wanted — and if so, use a passive pull-down/resistor network, never a GPIO output, per the datasheet's explicit constraint. |
| 8 | VBAT | I | Main module supply, **3.3–5.5V** (3.3V min / 4.2V typ / 5.5V max) | **Not yet wired.** Connect to this board's dedicated 5V rail — already sized for this exact module in `regulation.md` (LM1085-5.0, 2A budget, "5V rail (→ SA818S)"). Do not use the 3.3V rail; the 5V rail's current budget was specifically calculated around this module's TX current draw. |
| 9, 10 | GND | – | Ground | **Wired.** → `GND` |
| 11 | NC | – | Not connected | Leave open |
| 12 | ANT | I/O | RF signal, connect to 50Ω antenna | **Not yet wired.** Direct 50Ω trace to the SMA connector (Amphenol 132289, `C3172723`, same part now used for GPS's `J10`) — no bias-tee needed here (unlike GPS's active-antenna feed), this is a passive/direct antenna connection per the datasheet's own "connect 50 ohm antenna" description. |
| 13, 14, 15 | NC | – | Not connected | Leave open |
| 16 | RXD | I | Module UART RX (external TXD → here) | **Wired.** ← `SA818_UART_RXD` ← ESP32-S3 IO6 (verified correct crossover — MCU TX → module RX). **Datasheet caveat**: "before enter sleep mode, user need to pull low RXD pin to prevent current leakage or poor reset in the next time" — a firmware-level requirement (drive this line low before commanding the module to sleep), not a hardware change; worth flagging for firmware. |
| 17 | TXD | O | Module UART TX (→ external RXD) | **Wired.** → `SA818_UART_TXD` → ESP32-S3 IO7 (verified correct crossover — module TX → MCU RX). |
| 18 | MIC_IN | I | Microphone or line in — sized for mic-level signals, **not** line level (see `audio_ptt_path.md`'s attenuator writeup) | **Wired.** ← `SA818_MIC_IN` ← shared DAC/attenuator/jack circuit. |
| EP1, EP2 | — | Exposed pads on the `kicad_gx_library` footprint (`WIRELM-SMD_G-NICERF_SA818S-X`) | Not part of NiceRF's own functional pin table — these look like mechanical/shield-ground tabs, common on shielded RF modules of this type. | **Not yet wired.** Recommend tying to `GND` for proper shield grounding, but this is inferred from common practice for shielded-can modules, not confirmed by NiceRF's datasheet text directly — worth a quick sanity check against the footprint's own land-pattern documentation before committing. |

## 2. What's placed and verified correct so far

Traced directly against the live schematic (`SA818V.kicad_sch` + root-level cross-sheet
wiring), not assumed:

- **UART crossover**: correct. ESP32-S3 IO6 (`MCU_SA818_UART_TXD`) → module `RXD`;
  module `TXD` → ESP32-S3 IO7 (`MCU_SA818_UART_RXD`). Not a straight-through mistake.
- **PTT**: correct. Module `PTT` ← `SA818_PTT_IN` ← the shared PTT node driven by `Q10`
  (BSS138) and routed through the switched jack (`J11`) per `audio_ptt_path.md`.
- **MIC_IN**: correct. Module `MIC_IN` ← `SA818_MIC_IN` ← the shared DAC/attenuator/jack
  audio path.
- **GND**: correct, both ground pins tied.

## 3. Still open

- **`VBAT` unconnected** — needs the board's 5V rail. This is the biggest remaining gap;
  the module has no power yet.
- **`ANT` unconnected** — needs the 50Ω trace to the SMA connector (Amphenol 132289,
  matching `J10` on the GPS sheet).
- **`PD` unconnected** — needs a defined level (recommend tie high).
- **`H/L` unconnected** — needs a defined level (recommend leave open for high power;
  never tie to a GPIO-driven high per the datasheet's explicit warning).
- **`EP1`/`EP2` unconnected** — recommend `GND`, pending a quick footprint-documentation
  sanity check.
- **`AudioON`/`AF_OUT`** — deliberately left open this pass (no onboard speaker amp);
  already tracked as an open item in `audio_ptt_path.md`.
- **TX-safety timeout (74HC123) and PTT-keying optocoupler** — still not in this path;
  PTT currently runs directly from ESP32-S3 IO16 through `Q10`, no gating/isolation stage
  yet. Tracked in `open_items.md` and `ic_inventory.md`.

## 4. Reference documents

- **[NiceRF SA818S datasheet](https://logifind.com/u_file/2108/file/91fb17a776.pdf)**
  (Rev 1.3) — source for the full pin table, electrical specs (VBAT 3.3–5.5V, current
  consumption), and the PD/H-L/RXD-sleep caveats in this doc.
- [`audio_ptt_path.md`](audio_ptt_path.md) — the shared DAC/PTT subsystem feeding this
  module's `MIC_IN` and `PTT` pins.
- [`gps_path.md`](gps_path.md) — the sibling RF module sharing the same SMA connector
  part choice.
- [`regulation.md`](regulation.md) — the 5V rail this module's `VBAT` needs to land on.
