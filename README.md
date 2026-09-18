# Fox Hunt Controller — Hardware Design

Custom amateur radio fox hunt (hidden transmitter) controller, aimed at being a flexible, dynamic platform for a wide range of fox-hunt formats — not just a fixed single-mode beacon. Built around an ESP32-S3, u-blox GPS module (MAX-M10S), and SA818S 2m RF module, with a WiFi/BLE web UI for configuration and "found" logging, RTOS-based scheduling, RTC-driven wake and precise TX timing, GPS-disciplined timekeeping, and APRS output capability. Firmware developed in PlatformIO. Boards fabricated and assembled via JLCPCB (including their SMT assembly service). Planned as an iterative multi-revision build, proving out core software before locking down hardware.

**Note:** this initial revision is a dev/learning board — built to test out the design, get hands-on with the parts and tooling, and work out mistakes before committing to a more polished revision.

## Contents

- [Setup](#setup)
- [Power architecture overview](#power-architecture-overview) (diagram — full detail in `_docs/`)
- **[Full docs index →](_docs/README.md)** — hardware design detail, controller/peripheral platform decisions, BOM sourcing/datasheets, open items

## Setup

This project depends on three external symbol/footprint libraries, vendored as git submodules under `libs/` so the project is self-contained and doesn't require any machine-specific KiCad library setup. Project-level `sym-lib-table` and `fp-lib-table` files reference them via `${KIPRJMOD}`, so they resolve automatically once the submodules are checked out — no manual library registration needed.

Clone with submodules:

```sh
git clone --recurse-submodules https://github.com/gx1400/kicad_foxhunt_batterytest.git
```

Or, if already cloned:

```sh
git submodule update --init --recursive
```

Everything else (standard KiCad libraries: `Device`, `power`, `Resistor_SMD`, etc.) ships with any stock KiCad 10 install — no extra setup required.

## Power Architecture Overview

Two independent power sources feed the board, combined safely before regulation, then split into two independently-regulated rails:

```
18650 pack ──[MAX17320 protector]── +7.5V ──┐
                                              ├─[ORing FETs]── VRAW ──┬─[Buck+LDO]── 5V  (SA818S)
12–14.5V input ──[fuse]── +12V ──────────────┘                       └─[Buck+LDO]── 3.3V (ESP32-S3, GPS)
```

Full detail (battery protection, dual-input combining, regulation, the whole planned controller/peripheral platform, BOM sourcing, and open items) lives in **[`_docs/`](_docs/README.md)**.
