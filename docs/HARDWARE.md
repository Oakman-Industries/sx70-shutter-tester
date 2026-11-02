# Hardware reference

This page holds the circuit details and pin map for contributors. Start with the
[project overview](../README.md) or the [validation record](VALIDATION.md).
All capabilities remain untested; there is no working firmware release.

### Board at a glance

- 2-layer rigid PCB, **88 × 108 mm provisional outline** (marked *NOT FOR FABRICATION* on the
  silkscreen until it has been measured against real film packs and cameras)
- 158 machine-assembled SMD parts, all on the top side; nothing smaller than 0402
- The ESP32-S3-WROOM-1 module (U7) is soldered by hand, so it is left out of the assembly files
- Bottom side: only the two camera-contact pads (J6, Ø 14 mm, 35 mm apart) and 34 test points
- Battery cells are soldered directly to pads; there are no battery connectors

## Hardware capabilities and intended functions

### 1. Shutter timing: when does each part of the frame see light?

Nine **VBPW34S** silicon photodiodes (D5–D13) sit at the film plane in a 3 × 3 grid, named S1–S9:

```text
            rear of the film pack (top of the board)
      S7 (D11)          S8 (D12)          S9 (D13)
      S4 (D8)      LED (D4)  S5 (D9)      S6 (D10)
      S1 (D5)           S2 (D6)           S3 (D7)
            front / film-exit end (bottom of the board)
```

The columns are 34 mm apart. The outer sensors are intended to sample near the edges and
corners of the film window; their optical position still needs physical validation. S5 sits 7 mm below the centre to make room for the
inspection LED.

Each sensor has its **own comparator** (an LM393, U9–U17) and its **own ESP32 input**. When the
light on a sensor rises above a threshold, its output switches; when the light falls away, it
switches back. All nine comparator outputs are available in parallel. Firmware can use these to capture edges
across the nine positions and work out:

- how long light exceeds the threshold at each of the nine sampled positions;
- the order in which the sensors light up, which shows how the shutter blades open and close across
  the frame;
- whether all nine sampled positions receive light above the chosen threshold. Areas between
  sensors are not measured directly.

The threshold is shared by all nine comparators and set by the ESP32 (a filtered PWM output), so it
can be adjusted to the light source. Each comparator has a little hysteresis to keep its output
clean. Comparator delay, the sensor RC network, threshold, hysteresis and firmware timestamping all
contribute to the measured interval. **Shortest usable exposure and timing accuracy have not been
established.** A threshold-crossing interval is not automatically the equivalent full-open exposure.

### 2. Light intensity: can it build a coverage map?

**The hardware exposes an analog signal for every sensor, so relative intensity mapping is possible.** Every sensor's analog
signal also goes to the ESP32's analog-to-digital converter:

- **S5** (centre) has its own ADC input;
- **S1–S4 and S6–S9** share a second ADC input through an 8-channel analog switch (a CD4051, U8),
  which the ESP32 steps through one sensor at a time.

With a steady light in front of the lens, this gives a brightness value for each of the nine
positions, i.e. a coarse **nine-point map of illumination at the film plane**, useful for investigating
uneven exposure. Things to know:

- The values are **relative**, not calibrated light levels, and the nine sensors are not matched to
  each other yet. A calibration step in the firmware will be needed.
- Eight of the sensors are read **one after another**, not at the same instant. The achievable scan rate is not established; mux and ADC settling must be measured. A map is
  best attempted with steady illumination during a held-open/long exposure, or repeatable shots. The on/off timing in section 1 is not affected by
  this, because the comparators run in parallel.
- Bright light can saturate the analog signal and move comparator inputs outside their useful
  range; both intensity and timing behaviour need validation under the intended illumination.
- A brightness map and a timing map are different results. A per-point integrated exposure map
  requires sufficient samples through each exposure, calibration, and a repeatable light source;
  the current hardware/firmware has not demonstrated that.

### 3. Powering the camera, and cutting the power on purpose

An SX-70 is powered by the battery inside each film pack, through two contacts in the film chamber.
This board replaces that battery:

- a **boost converter** (TPS61288, U3) makes about **6.05 V** from the board's lithium cells;
- an **electronic fuse** (TPS259470, U4) sits between the converter and the camera. It provides overcurrent protection
  (nominal threshold about 3.3 A, provisional and not a guaranteed peak-current ceiling), disconnects the camera if the voltage
  goes above about 6.65 V, and reports a fault to the ESP32;
- a **10 mΩ current-sense resistor** (R21) then feeds the camera-contact pads on the underside (J6).

The ESP32 can control the camera power in two ways:

- **On/off** (`CAM_EN`): switches the whole 6 V output on or off.
- **Fast cut** (`CAM_CUT`): commands the eFuse output off without stopping the boost converter and can reconnect it. This is meant for **simulating power interruptions and power
  cycles**, e.g. a weak battery contact or a battery dropping out in the middle of a cycle, to see
  how the camera's electronics react. The actual voltage fall/rise depends on the output capacitors, camera load and eFuse start-up.
  There is no dedicated active discharge circuit, so a commanded cut is **not proof of a camera
  reset**. Firmware must check pad voltage and current and validate the required off-time.

The hardware interlock is intended to **disable camera power when valid USB input power is present**. Tests run on
the board's own cells, and results are meant to be read wirelessly.

What it **cannot** do: the camera voltage is fixed at about 6 V. An earlier version could lower the
voltage to simulate a tired battery; that circuit was removed to keep the board affordable.

### 4. Measuring the camera's current and voltage

| Measurement | Circuit / nominal conversion |
|---|---|
| **Camera current** | R21 10 mΩ + INA180A2 (U5), gain 50: `I_CAM ≈ 0.5 V per A` |
| **Camera-contact voltage** | R22/R24, 22 kΩ / 10 kΩ: `Vcamera ≈ Vadc × 3.2` |
| **Converter output voltage** | R25/R26, 200 kΩ / 100 kΩ: `Vboost ≈ Vadc × 3` |
| **Combined battery voltage** | R32/R33, 300 kΩ / 300 kΩ: `Vpack ≈ Vadc × 2` |

These are **nominal transfer ratios, not validated instrument ranges**. The usable range depends
on ESP32 ADC attenuation, calibration, amplifier headroom and component tolerances. The camera
output is designed around roughly 6 V; the divider ratios do not authorize higher camera voltages.

The intended result is a load profile showing motor/solenoid activity and voltage droop through a
camera cycle. These ADC channels are scanned, not sampled simultaneously. Capture rate, bandwidth,
noise, offset and calibration against a meter/scope remain unverified. The shunt measures the
**camera branch**, not the board's own current or the battery charge/discharge current.

### 5. Battery status and charging

- **Cells:** a single-cell-voltage lithium-ion (1S) pack charged to nominal 4.2 V. Two main pouch cells of about 20 × 50 mm (J2/J3) are
  connected in parallel and lie between the sensor columns. There is room for two more small
  optional cells (J4/J5) under the front lip. **No cell or parallel-pack assembly has been qualified.** The design assumes protected cells;
  cell matching, equalisation, protection compatibility and interconnection must be resolved before
  assembly. Fuses alone do not make an arbitrary group of pouch cells a suitable parallel pack. Every cell has its own fuse (F1–F4); F3/F4 are fitted
  only if the optional cells are used.
- **Charging:** through the **USB-C** port, with a BQ24074 linear charger (U2). The programmed nominal fast-charge current is about
  **270 mA** (3.3 kΩ ISET), with a nominal **410 mA input limit** (3.9 kΩ ILIM).
  These are resistor-derived targets, not measured limits. It uses 5 V USB input; there is no USB-PD
  fast-charge negotiation. Charging follows precharge, constant-current and constant-voltage phases;
  system load, input limits and thermal regulation can reduce the charge rate. The charger has power-path control: when USB is plugged in, the board runs from USB and the
  cells charge at the same time. Charging works whether the board is switched on or off.
- **What the ESP32 can see:** the combined pack voltage (useful for a rough state-of-charge estimate), whether the
  cells are **charging** (`CHG`), whether **valid USB input power is present** (`PGOOD`), and whether the camera
  output has tripped (`FLT`). An inactive `CHG` flag alone does not prove a full battery; it can
  also mean charging is absent, inhibited or faulted.
- **What it cannot see:** there is no fuel gauge (no counting of charge in and out) and **no battery
  temperature sensor**. All cells share one measured pack-voltage node; there are no individual-cell
  voltage, health, capacity or current measurements. A voltage-based percentage or runtime display
  would be an estimate, not a measured state of charge. The charger's temperature input is tied to a fixed resistor, so the board
  does not stop charging if the cells get hot. The charger's internal thermal regulation protects its die; it does not measure cell
  temperature. Cell protection circuits are not a substitute for cell-temperature supervision;
  this remains a battery-safety design decision to resolve.

### 6. Everything else

- **Inspection LED** (D4, white, centre of the board): shines out through the lens, so you can look
  into the front of the camera and see the shutter blades and aperture. Its brightness is set by the
  ESP32 (PWM), and the firmware is meant to limit how long it stays on.
- **ESP32-S3-WROOM-1** (U7, 8 MB flash): Wi-Fi and Bluetooth LE for a wireless user interface and
  data logging. Programmed over the USB-C port (native USB); BOOT and RESET buttons and UART test
  pads are there as a backup.
- **Status LED** (D3, red), and **34 labelled test points** on the underside for probing during
  bring-up.
- **3.3 V supply:** a TPS63802 buck-boost converter (U6), with a filtered analog branch for the
  sensors.

For component references and the dated evidence behind these descriptions, see
[validation and datasheet references](VALIDATION.md).

### ESP32 pin map (for future firmware)

| Function | GPIO |
|---|---|
| Light on/off, S1–S9 | IO11, IO12, IO13, IO14, IO16, IO17, IO38, IO39, IO47 |
| Shared comparator threshold (PWM) | IO15 |
| Sensor brightness: S5 direct / others via mux | IO8 / IO7 |
| Mux select A / B / C (CD4051 X0–X7 = S1, S2, S3, S4, S6, S7, S8, S9) | IO48 / IO10 / IO3 |
| Camera power on (`CAM_EN`) / fast cut (`CAM_CUT`) | IO18 / IO41 |
| Electronic-fuse fault (`FLT`, active low) | IO21 |
| Camera current (`I_CAM`) | IO2 |
| Camera-contact voltage / converter voltage / battery voltage | IO4 / IO9 / IO1 |
| Charging (`CHG`, active low) / USB power present (`PGOOD`, active low) | IO5 / IO6 |
| Inspection LED (PWM) / status LED | IO42 / IO40 |
| USB D− / D+ | IO19 / IO20 |

## Assembly-file scope

The BOM/CPL use JLCPCB-style assembly columns. This is an **unreleased development snapshot**, not an accepted fabrication package. The CAD source files are not
published. Left out of the BOM and CPL on purpose: the ESP32 module U7 (hand-soldered), the
not-fitted parts C17, C21, D2, F3 and F4, the power switch SW1 (no part chosen yet), the cell and
camera pads J2–J6, and the test points.

