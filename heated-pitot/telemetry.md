---
icon: sitemap
---

# Telemetry

The node publishes its status using the standard DroneCAN `uavcan.equipment.power.BatteryInfo` message at 5H&#x7A;**.** This message is for information only and not required for operation.&#x20;

> **We've adapted the battery message for our heated pitot purpose. The fields no longer reflect their original meanings. Doing this allows us to use a standard message, but does mean the fields require different interpretation to normal battery messages.**

### Field mapping

| BatteryInfo field     | Carries                                   | Units / encoding             |
| --------------------- | ----------------------------------------- | ---------------------------- |
| `temperature`         | internal pitot temperature                | Kelvin                       |
| `voltage`             | Heater supply voltage (to heater element) | Volts                        |
| `current`             | Heater current (to heater element)        | Amps                         |
| `state_of_health_pct` | **Heater state machine state**            | Enum (0–6) see section below |
| `state_of_charge_pct` | **Heater output power**                   | Watts                        |
| `battery_id`          | `DEVICE_ID` parameter                     | —                            |

All other `BatteryInfo` fields (remaining capacity, status flags, model name, etc.) are left at their defaults (zero/empty).

#### `state_of_health_pct` — heater state enum

The heater state-machine state is cast directly to an integer and placed in this field. Decode it as:

| Value | State          | Meaning                                                                                                                                                                                                                                                              |
| ----- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`   | `INITIALISE`   | Starting up / re-initialising.                                                                                                                                                                                                                                       |
| `1`   | `OFF`          | Heater commanded off (via actuator command).                                                                                                                                                                                                                         |
| `2`   | `FAULT`        | Recoverable fault detected (bad thermocouple reading, implausible rate-of-change, or suspected thermal runaway). Will retry normal operation after FAULT\_S                                                                                                          |
| `3`   | `HARD_FAULT`   | Terminal fault — only a power cycle resets this. The node also flags itself CRITICAL/OFFLINE and blinks its LED rapidly. This will occur if any of the onboard sensor circuits cannot be found on start-up. Likely the board has been damaged, replace heater board. |
| `4`   | `FAIR_WEATHER` | Idle: warm enough that heating isn't needed yet (only when `EN_FAIRWEATHER` is enabled).                                                                                                                                                                             |
| `5`   | `STANDBY`      | Reached if the heater element is too hot even at idle power. element is switched off until below target temperature. Likely only seen when stationary.                                                                                                               |
| `6`   | `STANDARD`     | standard operation heating toward `T_TARGET`.                                                                                                                                                                                                                        |



