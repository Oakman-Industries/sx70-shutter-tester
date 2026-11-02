# Development snapshot — 2 November 2026

**UNFINISHED / UNTESTED — NOT A FABRICATION RELEASE.** This record describes checks on the
saved design behind this snapshot. It is not proof of electrical function, mechanical fit,
battery safety or assembly acceptance. No hardware measurements are available.

## Evidence collected

| Check | Result | Scope / interpretation |
|---|---|---|
| Schematic netlist export | Completed | Saved schematic hierarchy; used to check the documented signal paths and GPIOs. |
| KiCad ERC | 0 errors, 0 warnings | Saved schematic; KiCad 10.0.8. Does not validate physical implementation. |
| KiCad DRC | 13 errors, 517 warnings, 0 unconnected items | Saved board, existing zone fills; no live save/refill was requested by this check. |
| Gerber/drill re-export | Completed | Two copper, two mask, two silk, outline, PTH and NPTH drill files, plus Gerber job metadata. |
| Geometry comparison | Matches prior public draft | Gerber/drill instructions match after ignoring export timestamps. Job metadata now records the intended black mask / white silk. |
| BOM/CPL reconciliation | 57 BOM rows, 158 parts; 158 CPL rows | Identical unique designator sets, no duplicates; all CPL placements are top-side. This is a structural check, not a physical rotation check. |
| Direct board render | Completed | Black presentation preset, component references/values hidden. Some component models are absent. |
| Physical build / scope / fit / thermal / RF tests | Not performed | No board has been built or tested. |
| Firmware | No usable release | An early core is parked, uncompiled and untested. No firmware is published here. |
| Gerber/drill viewer acceptance and assembler preview | Not completed | This snapshot is not an upload-ready order package. |

The archive is a development export. No design corrections or production silkscreen cleanup were
made for this publication. Hiding references in a render does not remove them from the Gerbers.

## DRC breakdown

| Severity | Rule | Count |
|---|---|---:|
| Error | Incomplete thermal-relief connection (`starved_thermal`) | 13 |
| Warning | Silkscreen overlap/clearance | 199 |
| Warning | Silkscreen over exposed copper/mask openings | 199 |
| Warning | Library footprint differs from placed footprint | 70 |
| Warning | Non-mirrored rear text | 37 |
| Warning | Silkscreen too close to board edge | 5 |
| Warning | Hole-to-hole spacing | 4 |
| Warning | Co-located drilled holes | 1 |
| Warning | Missing SW1 footprint (schematic parity) | 1 |
| Warning | J6 BOM-exclusion attribute differs (schematic parity) | 1 |

The warning total includes the two schematic-parity findings. Zero unrouted items means routing
connectivity is complete under the configured checks; **the DRC did not pass**. Thermal findings
recorded as accepted during development are not a fabrication waiver in this public snapshot.

## Release blockers

- Select and implement SW1. Its footprint is missing and the logic regulator's `PWR_EN` has no
  implemented switch drive on the PCB. A clean schematic ERC does not remove this blocker.
- Resolve or formally review DRC errors and warnings, including hole geometry, footprint changes,
  dense silkscreen and reversed rear text. Refill and rerun checks on the final saved revision.
- Measure the cartridge outline, component heights, sensor plane, pad position and camera-contact
  polarity against real hardware. The 88 × 108 mm outline and contact geometry are provisional.
- Qualify the selected parallel lithium-cell pack, protection, fuses, current capability, charge
  settings and thermal behaviour. There is no cell-temperature measurement in this design.
- Measure camera load, boost/eFuse behaviour and camera-pad discharge/recovery. A GPIO command
  to interrupt the supply does not prove the camera has lost power long enough to reset.
- Validate photodiode sensitivity, comparator input range, threshold/hysteresis, timing error,
  ADC/mux settling and sensor-to-sensor calibration. No shutter-speed accuracy is claimed.
- Implement and test working firmware and the wireless interface. Confirm RF operation inside
  the closed camera and account for acquisition timing while radios run.
- Check current component sourcing and all assembly rotations, solder paste requirements,
  polarity and exclusions in the assembler's preview. No order or stock acceptance is claimed.

## Component references

The project-specific claims come from exported connectivity and populated values. IC behaviour
and nominal calculations can be checked against the manufacturers' documents:

- [BQ24074 charger and power path](https://www.ti.com/lit/ds/symlink/bq24074.pdf): 4.2 V 1S charging,
  ISET/ILIM programming, status outputs, thermal regulation and TS input. R5 = 3.3 kΩ gives a nominal
  890 / 3300 ≈ 0.270 A fast-charge target; R4 = 3.9 kΩ gives a nominal 1610 / 3900 ≈ 0.413 A input
  limit. These calculations do not qualify this board or a cell pack.
- [INA180 current-sense amplifier](https://www.ti.com/lit/ds/symlink/ina180.pdf): A2 gain is 50 V/V;
  with the 10 mΩ shunt, the nominal output is 0.5 V/A.
- [TPS25947 eFuse](https://www.ti.com/lit/ds/symlink/tps25947.pdf): enable, overvoltage lockout,
  current protection, transient behaviour and start-up. The exact fitted variant is
  TPS259470LRPWR; board-level timing/current performance remains unmeasured.

Datasheets checked on 2 November 2026. A component capability is not a demonstrated system feature.
