# Power Architecture Overview

**Three** independent power sources feed the board — updated from an earlier two-source
version of this diagram once the USB/battery priority mux (Stages 2/3 below) was built:

1. **2S1P 18650 Li-ion pack** (4.2V/cell max, 8.4V max stack), protected and fuel-gauged by a MAX17320.
2. **External 12–14.5V input** (lead-acid or LiFePO4), arriving via Anderson Powerpoles, fused.
3. **USB VBUS** (via the CH334R hub's upstream port), only able to power the board when no battery is present — see [usb_battery_power_mux.md](usb_battery_power_mux.md).

Sources 1 and 2 are **fully isolated from each other on the charge path** — the 18650 pack is charged externally/separately, not through this circuit. This board's MAX17320 section handles protection and fuel gauging only. Their *outputs* are combined at Stage 1 below; USB is combined separately, downstream of regulation, at Stages 2/3.

```
18650 pack ──[MAX17320]── +7.5V ──┐
                                    ├─[Stage 1: TPS2121 VCOMP]── VRAW ──[soft-latch FET]──┬─[Buck+LDO]──[Stage 2: TPS2121]── SYSTEM_5V   (SA818S)
12–14.5V input ──[fuse]── +12V ────┘                                                      └─[Buck+LDO]──[Stage 3: TPS2121]── SYSTEM_3.3V (ESP32-S3, GPS)
                                                                                                              ↑                    ↑
                                                              USB VBUS (CH334R) ── USB_5V (raw) ──────────────┘                    │
                                                              USB VBUS (CH334R) ──[LM1117-3.3]── USB_3.3V ──────────────────────────┘
```

Stages 2/3 are set to **priority = battery** — a present, healthy battery always wins over
USB (e.g. a debug USB cable plugged in for flashing doesn't silently cut TX power). Details,
including the `ST`-pin USB-detect signal and its AND-gate combiner: [usb_battery_power_mux.md](usb_battery_power_mux.md).

The soft-latch stage (button/RTC-alarm/MCU-latch, gating VRAW before it reaches
regulation) is the mechanism behind the RTC-scheduled wake/sleep behavior described in
[controller_platform.md](controller_platform.md#power-sequencing--discrete-soft-latch) —
implemented, not just planned, as of the RTC peripheral work.

See:
- [Battery protection & fuel gauge (MAX17320)](battery_protection.md)
- [Dual-input power combining — Stage 1](power_input_combining.md)
- [Battery/USB power mux — Stages 2/3](usb_battery_power_mux.md)
- [Regulation — VRAW → 5V and 3.3V rails](regulation.md)
- [Controller & peripheral platform — soft-latch, RTC](controller_platform.md)
