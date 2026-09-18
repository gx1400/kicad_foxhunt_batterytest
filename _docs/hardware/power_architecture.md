# Power Architecture Overview

Two independent power sources feed the board, combined safely before regulation:

1. **2S1P 18650 Li-ion pack** (4.2V/cell max, 8.4V max stack), protected and fuel-gauged by a MAX17320.
2. **External 12–14.5V input** (lead-acid or LiFePO4), arriving via Anderson Powerpoles, fused.

These two sources are **fully isolated from each other on the charge path** — the 18650 pack is charged externally/separately, not through this circuit. This board's MAX17320 section handles protection and fuel gauging only. The two sources' *outputs* are combined downstream via ideal-diode ORing rather than sharing a charge path.

```
18650 pack ──[MAX17320 protector]── +7.5V ──┐
                                              ├─[ORing FETs]── VRAW ──┬─[Buck+LDO]── 5V  (SA818S)
12–14.5V input ──[fuse]── +12V ──────────────┘                       └─[Buck+LDO]── 3.3V (ESP32-S3, GPS)
```

See:
- [Battery protection & fuel gauge (MAX17320)](battery_protection.md)
- [Dual-input power combining](power_input_combining.md)
- [Regulation — VRAW → 5V and 3.3V rails](regulation.md)
