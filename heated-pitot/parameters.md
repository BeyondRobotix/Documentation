---
icon: wrench
---

# Parameters

| Parameter        | Type | Default | Min | Max  | Units    |
| ---------------- | ---- | ------- | --- | ---- | -------- |
| `NODEID`         | INT  | 100     | 1   | 127  | —        |
| `DEVICE_ID`      | INT  | 0       | 0   | 127  | —        |
| `T_THRESHOLD`    | INT  | 10      | 0   | 40   | °C       |
| `EN_FAIRWEATHER` | BOOL | 0       | 0   | 1    | —        |
| `P_MAX`          | INT  | 70      | 0   | 70   | W        |
| `CMD_CHANNEL`    | INT  | 10      | 0   | 100  | —        |
| `T_TARGET`       | INT  | 60      | 40  | 80   | °C       |
| `ASP_GATE`       | INT  | 15      | 0   | 100  | m/s      |
| `FAULT_S`        | INT  | 60      | 10  | 600  | s        |
| `TELEM_EN`       | BOOL | 1       | 0   | 1    | —        |
| `R_HEATER`       | INT  | 235     | 50  | 1000 | ×0.01 Ω  |
| `SELFTEST`       | BOOL | 0       | 0   | 1    | —        |

***

## Node / bus identity

#### `NODEID`

The DroneCAN node ID this device claims on the bus by default. Will respect DNA allocation from the autopilot if this ID is already taken. **Changing this only takes effect after the node is restarted.**

* Default: `100` · Range: `1`–`127`

#### `DEVICE_ID`

Battery / device identifier reported in the published `BatteryInfo.battery_id` field. Use it to tell multiple heated-pitot nodes apart in the autopilot's battery monitor / logs. Has no effect on control behaviour.

* Default: `0` · Range: `0`–`127`

***

## Command interface

#### `CMD_CHANNEL`

The actuator channel (`actuator_id`) this node listens to in an `uavcan.equipment.actuator.ArrayCommand`. The autopilot uses this channel to turn the heater **on** or **off**:

* **PWM command type:** value `< 500` is ignored (no change); `≥ 1500` = ON (arm), otherwise OFF.
* **Unitless command type:** value must be within `-1…1`; `≥ 0.75` = ON, `≤ 0.25` = OFF, in-between = no change.

If no actuator command is received for 5 s while the heater is OFF, the node fails **safe to ON** (re-initialises the heater) so a lost link doesn't leave the pitot unheated.

* Default: `10` · Range: `0`–`100`

***

## Temperature control

#### `T_TARGET`

The heater temperature setpoint the PID loop drives toward, in °C. This is the regulated operating temperature of the probe.

We currently recommend at least 60 degrees to ensure ice is melted quickly from the probe, however, with further testing Beyond Robotix may be able to recommend a reduced temperature to save power.

* Default: `60` °C · Range: `40`–`80` °C

#### `T_THRESHOLD`

Fair-weather activation threshold, in °C. While `EN_FAIRWEATHER` is enabled, the heater stays idle until the measured temperature drops **below** this value (or fair-weather is disabled), at which point it moves into active heating.

* Default: `10` °C · Range: `0`–`40` °C

#### `EN_FAIRWEATHER`

Enables fair-weather idling. When `1`, the heater sits idle (in `FAIR_WEATHER`) until it's cold enough (see `T_THRESHOLD`) to be worth heating, saving power in warm conditions. When `0`, the heater skips fair-weather and always progresses to active temperature regulation.

* Default: `0` (disabled) · Range: `0`–`1`

***

## Power / regulator limits

#### `P_MAX`

Maximum heater power, in watts. This caps the upper limit of the PID output and therefore the most power the regulator will deliver to the heating element.

* Default: `70` W · Range: `0`–`70` W

> **Note:** input voltage below \~12 V reduces the regulator's achievable maximum output regardless of this setting; the node emits a `Vin < 12V` warning when that happens.

***

## Fault detection

#### `ASP_GATE`

Airspeed gate for thermal-runaway detection, in m/s. The "probe fell off / thermal runaway" fault (sustained max power with no temperature rise) is **suppressed when airspeed is at or above this value**, because at high airspeed the probe can legitimately draw full power without heating up. Below this airspeed, the runaway check is active.

* Default: `15` m/s · Range: `0`–`100` m/s

#### `FAULT_S`

Fault cooldown, in seconds. After a thermal-runaway fault, the node holds in `FAULT` for this long before retrying. (Sensor-blip faults such as NaN readings or over-fast changes retry immediately; only thermal-runaway uses this cooldown.)

* Default: `60` s · Range: `10`–`600` s

***

## Telemetry

#### `R_HEATER`

Resistance of the heating element, in hundredths of an ohm (so `206` means 2.06 Ω). The node has no current sensor: it measures the heater output voltage on its ADC divider and infers current and power from this value via Ohm's law (`I = V / R`, `P = V × I`). Only the `current` field of the telemetry message and the serial debug output depend on it — **heater control is unaffected**, so mis-setting it cannot make the heater behave badly.

The default is the measured element resistance. If you meter your own element cold, enter that value instead. Expect the inferred current to read a few percent high when the element is hot, since the parameter is a fixed resistance and the element's resistance rises with temperature.

* Default: `235` (2.35 Ω) · Range: `50`–`1000` (0.5–10.0 Ω)

#### `TELEM_EN`

Enables publishing of the node's `uavcan.equipment.power.BatteryInfo` status message. When `1`, the node broadcasts heater telemetry (heater voltage, inferred current, temperature, heater state, output power) every heater cycle. When `0`, the message is suppressed — useful to reduce bus traffic when the telemetry isn't needed. Disabling telemetry has no effect on heater control.

* Default: `1` (enabled) · Range: `0`–`1`

***

## Diagnostics

#### `SELFTEST`

Write `1` to run the hardware self-test, which drives the trim DAC and the enable line through a short sequence (\~4.5 s, under \~15 J into the element) and checks the regulator responds and tracks. The node writes the parameter back to `0` when the run ends, including when it refuses to start.

Track progress with the `SELFTEST_ACTIVE` bit in `NodeStatus`'s `vendor_specific_status_code`, not by polling this parameter. The result appears in the same field: `SELFTEST_PASSED` for a clean run, or the specific failure bits.

The test refuses to run unless the heater supply is present, the temperature is below target and finite, airspeed is under 1 m/s, and the node is not in `HARD_FAULT`. A refused or aborted run reports `SELFTEST_INCONCLUSIVE`.

* Default: `0` · Range: `0`–`1`

***

## Quick reference: worked examples

* **Regulate to 65 °C, never exceed 40 W** → `T_TARGET = 65`, `P_MAX = 40`.
* **Always heat (no fair-weather idling)** → `EN_FAIRWEATHER = 0`.
* **Listen to autopilot actuator channel 3** → `CMD_CHANNEL = 3`.
