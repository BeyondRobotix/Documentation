# Parameters

| Parameter        | Type | Default | Min | Max  | Units |
| ---------------- | ---- | ------- | --- | ---- | ----- |
| `NODEID`         | INT  | 100     | 1   | 127  | —     |
| `DEVICE_ID`      | INT  | 0       | 0   | 127  | —     |
| `T_THRESHOLD`    | INT  | 10      | 0   | 40   | °C    |
| `EN_FAIRWEATHER` | BOOL | 0       | 0   | 1    | —     |
| `P_MAX`          | INT  | 70      | 0   | 70   | W     |
| `CMD_CHANNEL`    | INT  | 10      | 0   | 100  | —     |
| `KP`             | INT  | 100     | 0   | 1000 | ×0.01 |
| `KI`             | INT  | 50      | 0   | 1000 | ×0.01 |
| `KD`             | INT  | 0       | 0   | 1000 | ×0.01 |
| `T_TARGET`       | INT  | 60      | 40  | 80   | °C    |
| `CHANGE_LIMIT`   | INT  | 10      | 0   | 100  | °C/s  |
| `RAMP_S`         | INT  | 5       | 0   | 30   | s     |
| `MIN_POWER`      | INT  | 2       | 0   | 10   | W     |
| `ASP_GATE`       | INT  | 15      | 0   | 100  | m/s   |
| `FAULT_S`        | INT  | 60      | 10  | 600  | s     |

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

#### `MIN_POWER`

Low-power cutoff, in watts. When the PID demand falls below this value the heater is switched fully off, avoiding the regulator's \~5 W minimum-output floor from overshooting the target at low demand. Hysteresis re-enables the heater once demand rises back above `2 × MIN_POWER`. Set to `0` to disable the cutoff.

* Default: `2` W · Range: `0`–`10` W

#### `RAMP_S`

Soft-start ramp time, in seconds. On entry to active PID control (`STANDARD`), the PID output upper limit is ramped from 0 up to `P_MAX` over this many seconds, so the heater eases into full power rather than slamming on. Set to `0` for no ramp.

* Default: `5` s · Range: `0`–`30` s

***

## PID gains

The PID loop maps heater temperature error to a power demand (0…`P_MAX` W).

> **Scaling:** `KP`, `KI`, and `KD` are transmitted as integers scaled by ×100. The firmware multiplies by 0.01 on receipt, so a parameter value of `30` means an actual gain of **0.30**. To set Kp = 0.30, write `KP = 30`.

#### `KP`

Proportional gain (×0.01). Parameter value `100` → Kp = 1.00.

* Default: `100` (Kp = 1.00) · Range: `0`–`1000` (Kp = 0.00…10.00)

#### `KI`

Integral gain (×0.01). Parameter value `50` → Ki = 0.50.

* Default: `50` (Ki = 0.50) · Range: `0`–`1000` (Ki = 0.00…10.00)

#### `KD`

Derivative gain (×0.01). Parameter value `0` → Kd = 0.00.

* Default: `0` (Kd = 0.00) · Range: `0`–`1000` (Kd = 0.00…10.00)

***

## Fault detection

#### `CHANGE_LIMIT`

Thermocouple rate-of-change fault threshold, in °C per second. If the measured temperature jumps faster than this between cycles, the reading is treated as implausible and the heater trips into `FAULT` (typically indicates a disconnected or shorting thermocouple).

* Default: `10` °C/s · Range: `0`–`100` °C/s

#### `ASP_GATE`

Airspeed gate for thermal-runaway detection, in m/s. The "probe fell off / thermal runaway" fault (sustained max power with no temperature rise) is **suppressed when airspeed is at or above this value**, because at high airspeed the probe can legitimately draw full power without heating up. Below this airspeed, the runaway check is active.

* Default: `15` m/s · Range: `0`–`100` m/s

#### `FAULT_S`

Fault cooldown, in seconds. After a thermal-runaway fault, the node holds in `FAULT` for this long before retrying. (Sensor-blip faults such as NaN readings or over-fast changes retry immediately; only thermal-runaway uses this cooldown.)

* Default: `60` s · Range: `10`–`600` s
