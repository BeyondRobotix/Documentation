---
icon: sitemap
---

# Telemetry

The node publishes its status using the standard DroneCAN `uavcan.equipment.power.BatteryInfo` message, broadcast every heater cycle (\~200 ms, i.e. **5 Hz**) at medium transfer priority. This message is for information only and not required for operation. Publishing can be turned off entirely with the `TELEM_EN` parameter.

> **We've adapted the battery message for our heated pitot purpose. The fields no longer reflect their original meanings. Doing this allows us to use a standard message, but does mean the fields require different interpretation to normal battery messages.** In particular, `state_of_health_pct` carries the **heater state**, and `state_of_charge_pct` carries the **output power in watts**.

### Field mapping

| BatteryInfo field     | Carries                                   | Units / encoding             | How to interpret                                                  |
| --------------------- | ----------------------------------------- | ---------------------------- | ----------------------------------------------------------------- |
| `temperature`         | Heater (thermocouple) temperature         | Kelvin                       | Subtract **273** to get °C.                                       |
| `voltage`             | Heater output voltage (to heater element) | Volts                        | As-is. Measured directly on the board's ADC divider.              |
| `current`             | Heater current (**inferred**)             | Amps                         | Not measured. Computed as `voltage / R_HEATER` — see below.       |
| `state_of_health_pct` | **Heater state machine state**            | Enum (0–9) see section below | Decode with the state table below. **Not** a health percentage.   |
| `state_of_charge_pct` | **Heater output power**                   | Watts                        | Integer watts (0…`P_MAX`). **Not** a charge percentage.           |
| `battery_id`          | `DEVICE_ID` parameter                     | —                            | Identifies which node this is; set via the `DEVICE_ID` parameter. |

All other `BatteryInfo` fields (remaining capacity, status flags, model name, etc.) are left at their defaults (zero/empty) and carry **no meaning** — do not read anything into them.

#### `temperature` — °C conversion

The firmware sends `heater_temperature + 273`, so:

```
heater_temp_°C = temperature_field − 273
```

(Note it adds a flat 273, not 273.15, so expect up to \~0.15 °C of rounding offset versus a true Kelvin→Celsius conversion — negligible for monitoring.)

#### `current` — inferred, not measured

The node carries no current sensor. It samples the heater output voltage on its ADC divider and applies Ohm's law using the `R_HEATER` parameter (heater resistance in hundredths of an ohm, default `235` = 2.35 Ω, the measured element resistance):

```
current_A = voltage_V / (R_HEATER / 100)
```

Two consequences worth knowing before you trust the number:

* **It is only as good as `R_HEATER`.** If the element in your probe isn't the nominal one, set the parameter to its measured cold resistance.
* **It reads high when hot.** The element's resistance rises with temperature while `R_HEATER` is fixed, so real current at operating temperature is somewhat lower than reported.

Use it for trend and sanity-checking (is the heater drawing roughly what the commanded power implies?), not for energy accounting.

#### `state_of_health_pct` — heater state enum

The heater state-machine state is cast directly to an integer and placed in this field. Decode it as:

| Value | State          | Meaning                                                                                                                                                                                                                                                            |
| ----- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`   | `INITIALISE`   | Starting up / re-initialising: clearing buffers, configuring the PID controller.                                                                                                                                                                                   |
| `1`   | `OFF`          | Heater commanded off (via actuator command).                                                                                                                                                                                                                       |
| `2`   | `FAULT`        | Recoverable fault detected (bad thermocouple reading, implausible rate-of-change, or suspected thermal runaway). Will retry normal operation after `FAULT_S`.                                                                                                       |
| `3`   | `HARD_FAULT`   | Terminal fault — only a power cycle resets this. The node also flags itself CRITICAL/OFFLINE and blinks its LED rapidly. This will occur if any of the onboard sensor circuits cannot be found on start-up. Likely the board has been damaged, replace heater board. |
| `4`   | `FAIR_WEATHER` | Idle: warm enough that heating isn't needed yet (only when `EN_FAIRWEATHER` is enabled).                                                                                                                                                                            |
| `5`   | `STANDBY`      | Not entered. Retained so the enum numbering stays stable; a node in it is redirected to `CONTROL`.                                                                                                                                                                  |
| `6`   | `STANDARD`     | Not entered. As above.                                                                                                                                                                                                                                             |
| `7`   | `UNPOWERED`    | No heater supply on Vin. The node still runs off the bus rail, so it stays fully alive on CAN.                                                                                                                                                                      |
| `8`   | `SELFTEST`     | Hardware self-test running. `state_of_charge_pct` reads 0 throughout, even though the heater is being driven.                                                                                                                                                       |
| `9`   | `CONTROL`      | Active closed-loop heating toward `T_TARGET`.                                                                                                                                                                                                                      |

Normal warmed-up operation sits in **CONTROL (9)**. Seeing **FAULT (2)** or **HARD\_FAULT (3)** indicates a problem — most often a thermocouple that has come loose or a probe that has physically detached. **UNPOWERED (7)** means the heater supply is missing, not that the node has failed.

#### `state_of_charge_pct` — output power (W)

The heater's commanded output power (the PID demand, 0…`P_MAX` watts) is cast to an integer and placed here. So a value of `35` means the heater is currently being driven at \~35 W. It is **not** a percentage and does not map to 0–100.

> Because this field is a `uint7` (0–127) in the standard message, and `P_MAX` maxes out at 70 W, the value comfortably fits. Read it as raw watts.

### Reading it in a ground station

Most GCS software will display this as a "battery." To make sense of it:

* **Temperature** — displayed in Kelvin by many tools; subtract 273 for °C.
* **Voltage** — the heater output voltage (not a battery).
* **Current** — an estimate, not a measurement; see the note above.
* **State of Health** — ignore the "%" label; read the number against the **state table** above.
* **State of Charge** — ignore the "%" label; read the number as **watts of heater power**.
* **Battery ID** — matches the node's `DEVICE_ID` parameter, letting you tell multiple heated-pitot nodes apart.

### Worked example

A received `BatteryInfo` of:

```
temperature         = 333.0   (K)
voltage             = 13.6    (V)
current             = 5.8     (A, inferred)
state_of_health_pct = 9
state_of_charge_pct = 35
battery_id          = 0
```

means: heater at **60 °C** (333 − 273), driven at **13.6 V** (≈ **5.8 A** inferred through a 2.35 Ω element), in the **CONTROL** state (active closed-loop heating), currently driving **35 W**, reported by the node whose `DEVICE_ID` is **0**.
