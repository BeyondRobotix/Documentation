---
icon: gear
---

# Advanced Configuration

## BR Core — Interface Reference

### Overview

The BR Core is a compact STM32H743-based autopilot designed for integration into custom UAV platforms. It is built around a three-layer stack — processor, power, and connector — with the default connector layer providing a set of JST-GH style breakouts. Custom connector layers can be designed to re-route the same signals to any layout.

All digital I/O passes through SN74LVC8T245 level-shifting buffers on the power layer. CAN and Ethernet differential pairs are direct pass-throughs (no level shifting). PWM outputs are selectable between 3.3 V and 5 V via a software-controlled GPIO (see [PWM voltage selection](#pwm-voltage-selection)).

***

### Physical connectors (default connector layer)

The following connectors are present on the default connector layer. All use the JST-GH 1.25 mm pitch format unless stated otherwise. Pin 1 is marked by a small triangle or dot on the connector housing.

All UART connectors follow the standard ArduPilot pinout: **5V · TX · RX · GND** for simple ports, and **5V · TX · RX · CTS · RTS · GND** for ports with hardware flow control.

#### RC (Serial1 — USART1)

Default protocol: RC input (SBUS/CRSF/etc.)\
ArduPilot parameter: `SERIAL1_PROTOCOL`

| Pin | Signal | Direction | Voltage |
| --- | ------ | --------- | ------- |
| 1   | 5V     | Output    | 5 V     |
| 2   | TX1    | Output    | 3.3 V   |
| 3   | RX1    | Input     | 3.3 V   |
| 4   | GND    | —         | 0 V     |

> USART1 does not have hardware flow control pins on this connector. It is the recommended port for RC receivers.

***

#### TELEM1 (Serial2 — USART2)

Default protocol: MAVLink 2\
ArduPilot parameter: `SERIAL2_PROTOCOL`\
Power rail: Telem 5 V (current-limited, see [Power](#power))

| Pin | Signal     | Direction | Voltage |
| --- | ---------- | --------- | ------- |
| 1   | Telem 1 5V | Output    | 5 V     |
| 2   | TX2        | Output    | 3.3 V   |
| 3   | RX2        | Input     | 3.3 V   |
| 4   | CTS2       | Input     | 3.3 V   |
| 5   | RTS2       | Output    | 3.3 V   |
| 6   | GND        | —         | 0 V     |

> This is the primary telemetry port. Connect a radio modem (e.g. SiK, Herelink Air Unit) here. The 5 V rail on this connector is sourced from a dedicated current-limited supply separate from the general peripherals rail.

***

#### SERIAL3 (USART3)

Default protocol: None (user-defined)\
ArduPilot parameter: `SERIAL3_PROTOCOL`

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX3            | Output    | 3.3 V   |
| 3   | RX3            | Input     | 3.3 V   |
| 4   | CTS3           | Input     | 3.3 V   |
| 5   | RTS3           | Output    | 3.3 V   |
| 6   | GND            | —         | 0 V     |

***

#### SERIAL4 (UART4)

Default protocol: None (user-defined)\
ArduPilot parameter: `SERIAL5_PROTOCOL`

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX4            | Output    | 3.3 V   |
| 3   | RX4            | Input     | 3.3 V   |
| 4   | GND            | —         | 0 V     |

***

#### SERIAL5 (UART5)

Default protocol: None (user-defined)\
ArduPilot parameter: `SERIAL6_PROTOCOL`

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX5            | Output    | 3.3 V   |
| 3   | RX5            | Input     | 3.3 V   |
| 4   | GND            | —         | 0 V     |

***

#### SERIAL6 (USART6)

Default protocol: None (user-defined)\
ArduPilot parameter: `SERIAL7_PROTOCOL`

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX6            | Output    | 3.3 V   |
| 3   | RX6            | Input     | 3.3 V   |
| 4   | GND            | —         | 0 V     |

***

#### GPS (Serial4 — UART7)

Default protocol: GPS\
ArduPilot parameter: `SERIAL4_PROTOCOL`\
This port also carries I²C for an external compass.

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX7            | Output    | 3.3 V   |
| 3   | RX7            | Input     | 3.3 V   |
| 4   | SCL            | Output    | 3.3 V   |
| 5   | SDA            | Output    | 3.3 V   |
| 6   | GND            | —         | 0 V     |

> This is the recommended GPS port. Connect a u-blox M8N/M9N/F9P or compatible module here. The I²C lines are for a compass co-located on the GPS module (e.g. IST8310, QMC5883). Set `COMPASS_TYPEMASK` to exclude any compasses you are not using.

***

#### CAN1

Termination: 120 Ω installed on the connector layer.

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | CAN1\_H        | Bidir     | —       |
| 3   | CAN1\_L        | Bidir     | —       |
| 4   | GND            | —         | 0 V     |

> CAN1 maps to FDCAN1 on the STM32H743. Use this for DroneCAN peripherals (GPS, ESCs, airspeed sensors, power monitors, etc.). ArduPilot parameters: `CAN_P1_DRIVER = 1`, `CAN_D1_PROTOCOL = 1`.

***

#### CAN2

Termination: 120 Ω installed on the connector layer.

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | CAN2\_H        | Bidir     | —       |
| 3   | CAN2\_L        | Bidir     | —       |
| 4   | GND            | —         | 0 V     |

> CAN2 maps to FDCAN2. Parameters: `CAN_P2_DRIVER = 1`, `CAN_D2_PROTOCOL = 1`.

***

#### I²C

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | SCL            | Output    | 3.3 V   |
| 3   | SDA            | Output    | 3.3 V   |
| 4   | GND            | —         | 0 V     |

> I²C bus 2 (I2C4 on STM32H743). Suitable for external compasses, barometers, or airspeed sensors that require I²C. The bus is buffered on the power layer.

***

#### ETH (Ethernet)

| Pin | Signal | Direction | Voltage |
| --- | ------ | --------- | ------- |
| 1   | TX\_P  | Output    | —       |
| 2   | TX\_N  | Output    | —       |
| 3   | RX\_P  | Input     | —       |
| 4   | RX\_N  | Input     | —       |

> 100BASE-TX Ethernet via LAN8742A PHY. No power pins are present on this connector — power your Ethernet device separately. The Ethernet interface is enabled automatically at boot and requires no ArduPilot parameter changes. Use for MAVLink 2 over UDP to a companion computer or ground station on the same network. The ETH\_LED output is available on the DEBUG connector.

***

#### USB

| Pin | Signal        | Direction | Voltage |
| --- | ------------- | --------- | ------- |
| 1   | Pwr\_USB2\_In | Input     | 5 V     |
| 2   | USB\_P        | Bidir     | —       |
| 3   | USB\_N        | Bidir     | —       |
| 4   | GND           | —         | 0 V     |

> This is the primary USB port (OTG1, USB FS). Connect to a ground control station for configuration and firmware updates. When powered via USB only, the board will boot but ESC/servo power will not be available.

***

#### DEBUG

14-pin, 1.27 mm pitch (FTSH-107-01-L-DV-K, compatible with standard ARM SWD cables).

| Pin | Signal   | Notes                                      |
| --- | -------- | ------------------------------------------ |
| 1   | 3.3V     | From processor layer LDO                   |
| 2   | GND      |                                            |
| 3   | SWDIO    | SWD data                                   |
| 4   | GND      |                                            |
| 5   | SWCLK    | SWD clock                                  |
| 6   | GND      |                                            |
| 7   | TX8      | UART8 TX (debug console, 57600 baud)       |
| 8   | GND      |                                            |
| 9   | RX8      | UART8 RX                                   |
| 10  | GND      |                                            |
| 11  | GPIO1    | PJ2 (PINIO1, GPIO 81)                      |
| 12  | PWM10    | TIM3\_CH3                                  |
| 13  | ETH\_LED | Ethernet link/activity (resistor required) |
| 14  | GND      |                                            |

> UART8 is the ArduPilot debug console (`STDOUT_SERIAL SD8`, 57600 baud). Connect a 3.3 V FTDI cable to TX8/RX8 to view boot output and ArduPilot log messages. It is also exposed as Serial8 (`SERIAL8_PROTOCOL`) if you wish to use it for a MAVLink link or other purpose.

> The SWD interface is used for firmware flashing and debugging with tools such as ST-Link, J-Link, or Black Magic Probe. The bootloader is at the start of flash and can also be entered via the ArduPilot `BL_UPDATE` parameter.

***

#### PWR1 / PWR2

6-pin power input connectors. PWR1 and PWR2 provide independent redundant power paths — either connector alone is sufficient to power the board.

| Pin | Signal  | Direction | Voltage    |
| --- | ------- | --------- | ---------- |
| 1   | Pwr\_In | Input     | 4.2–5.75 V |
| 2   | Pwr\_In | Input     | 4.2–5.75 V |
| 3   | CAN2\_H | Bidir     | —          |
| 4   | CAN2\_L | Bidir     | —          |
| 5   | GND     | —         | 0 V        |
| 6   | GND     | —         | 0 V        |

> The power connectors also carry CAN2 as a convenience for power modules that include a DroneCAN interface (e.g. Zubax Myxa, Pomegranate Systems PM). If you are not using a DroneCAN power module, leave CAN2\_H/L unconnected.

***

#### PWM outputs

The 16 PWM outputs are available on a 3×16 row of 2.54 mm pitch through-holes at the bottom edge of the board, with three rows labelled **PWM**, **Spwr** (servo power), and **GND**.

| Row  | Function                                                                                                    |
| ---- | ----------------------------------------------------------------------------------------------------------- |
| PWM  | Signal (level-shifted, 3.3 V or 5 V — see below)                                                            |
| Spwr | Servo rail power (connected to SERVO\_VBUS\_ADC sense line, **not a power output** — requires external BEC) |
| GND  | Ground                                                                                                      |

PWM outputs 1–8 are the **MAIN** outputs and pass through the level shifter (voltage-selectable). PWM outputs 9–18 are **AUX** outputs and are direct 3.3 V GPIO.

| Output | Timer channel | ArduPilot function |
| ------ | ------------- | ------------------ |
| PWM1   | TIM1\_CH1     | MAIN 1             |
| PWM2   | TIM1\_CH2     | MAIN 2             |
| PWM3   | TIM1\_CH3     | MAIN 3             |
| PWM4   | TIM1\_CH4     | MAIN 4             |
| PWM5   | TIM2\_CH1     | MAIN 5             |
| PWM6   | TIM2\_CH3     | MAIN 6             |
| PWM7   | TIM2\_CH4     | MAIN 7             |
| PWM8   | TIM3\_CH1     | MAIN 8             |
| PWM9   | TIM3\_CH2     | AUX 1              |
| PWM10  | TIM3\_CH3     | AUX 2              |
| PWM11  | TIM3\_CH4     | AUX 3              |
| PWM12  | TIM4\_CH2     | AUX 4              |
| PWM13  | TIM4\_CH3     | AUX 5              |
| PWM14  | TIM4\_CH4     | AUX 6              |
| PWM15  | TIM5\_CH1     | AUX 7              |
| PWM16  | TIM5\_CH2     | AUX 8              |
| PWM17  | TIM5\_CH3     | AUX 9              |
| PWM18  | TIM5\_CH4     | AUX 10             |

***

### Power

#### Input

The board accepts 4.2–5.75 V on PWR1 and/or PWR2. Both inputs feed the internal 5 V bus through independent current-limited load switches with power-good monitoring. Either input alone is sufficient; there is automatic failover with no configuration required.

A third 5 V input (`Pwr_USB2_In`) is available on the USB connector and on J2 of the main stack, intended for USB power from a carrier board.

#### Regulated outputs

| Rail           | Voltage | Source                                          | Notes               |
| -------------- | ------- | ----------------------------------------------- | ------------------- |
| Peripherals 5V | 5 V     | Internal, current-limited (2 A, 16 V rated)     | General peripherals |
| Telem 1 5V     | 5 V     | Internal, current-limited (separate from above) | Telemetry radio     |
| 3.3V           | 3.3 V   | AP2112K LDO on processor layer                  | Logic supply        |

> **Do not exceed 2 A total** on the Peripherals 5V rail across all connectors combined. The Telem 1 5V rail has its own 2 A limit. Both rails have power-fault signals monitored by the MCU (`VDD_5V_PERIPH_nOC` on PI4, `VDD_5V_HIPOWER_nOC` on PI5).

#### Power monitoring

Four current sense channels are available for ArduPilot battery monitor instances:

| ArduPilot instance | MCU pin     | Sense point             | hwdef name           |
| ------------------ | ----------- | ----------------------- | -------------------- |
| BATT5              | PC3 (ADC1)  | PWR1 input current      | `BATT5_CURRENT_SENS` |
| BATT6              | PF11 (ADC1) | PWR2 input current      | `BATT6_CURRENT_SENS` |
| BATT7              | PA6 (ADC1)  | USB power layer current | `BATT7_CURRENT_SENS` |
| BATT8              | PF13 (ADC2) | USB carrier current     | `BATT8_CURRENT_SENS` |

These measure current only. The board does not have direct access to raw battery voltage — voltage monitoring should be done externally via a DroneCAN power module, or via a resistor divider on ADC7 or ADC8 (J2 pins 67 and 57).

A 5 V bus sense line (`VDD_5V_SENS` on PF3, ADC3) monitors the internal 5 V rail through a 1:1 divider and is configured with `SCALE(2)`.

#### Servo rail

The servo power rail (Spwr row on the PWM header) is **not powered by the autopilot**. Connect an external BEC to the Spwr and GND pins to power your servos. The servo rail voltage is monitored by the MCU via `FMU_SERVORAIL_VCC_SENS` (PC2, ADC1, 27k/3.3k divider) and reported as `BATT_VOLT_MULT`. The default scale factor in the hwdef is 8.18 — verify this against your actual resistor values.

To set up the servo rail voltage monitor in ArduPilot:

```
BATT_MONITOR = 3       # Analog voltage only
BATT_VOLT_PIN = 13     # PC2 ADC channel
BATT_VOLT_MULT = 8.18
```

***

### PWM voltage selection

PWM outputs 1–8 (MAIN) pass through level-shifting buffers whose output voltage is software-selectable between 3.3 V and 5 V. The default in the firmware is **3.3 V**.

To switch to 5 V output (required for most standard RC servos):

```
SERVO_BLH_POLES = 0    # not related, just context
# Set GPIO 3 HIGH for 5V, LOW for 3.3V
```

In ArduPilot, control this via the `HAL_GPIO_PWM_VOLT_PIN` (GPIO 3, PI6):

```python
# From GCS or scripting:
param set BRD_PWM_VOLT_EN 1     # if available in your build
```

Or directly via the `relay` interface:

```
RELAY1_PIN = 3        # GPIO 3 = PWM voltage selector
RELAY1_DEFAULT = 1    # 1 = HIGH = 5V output
```

Then use `do-set-relay` in a mission or MAVLink command to toggle. Set to HIGH (1) for 5 V servo output, LOW (0) for 3.3 V.

> PWM outputs 9–18 (AUX) are direct GPIO outputs and are always 3.3 V regardless of this setting.

***

### Relay / GPIO outputs

Three software-controlled GPIO outputs are available for relay or payload control, accessible on custom connector layer designs (not broken out on the default connector layer).

| ArduPilot name | MCU pin | GPIO number | hwdef        |
| -------------- | ------- | ----------- | ------------ |
| PINIO1         | PJ2     | 81          | `PJ2 PINIO1` |
| PINIO2         | PJ5     | 82          | `PJ5 PINIO2` |
| PINIO3         | PJ6     | 83          | `PJ6 PINIO3` |

To configure as relays:

```
RELAY1_PIN = 81    # PINIO1
RELAY2_PIN = 82    # PINIO2
RELAY3_PIN = 83    # PINIO3
```

Two additional high-current enable outputs control the 5V power rails and can also be used as relays if power switching is needed:

| Function                         | MCU pin | GPIO number | Default   |
| -------------------------------- | ------- | ----------- | --------- |
| HI\_PWR\_EN (Telem 5V)           | PJ0     | 85          | HIGH (on) |
| PERIPH\_PWR\_EN (Peripherals 5V) | PJ1     | 86          | HIGH (on) |

> **Caution:** disabling PERIPH\_PWR\_EN (GPIO 86) will cut power to all peripherals on the Peripherals 5V rail. Only use this as a relay if you understand the downstream effects.

***

### Serial port assignments

| Serial  | Hardware   | Default protocol | Connector                 |
| ------- | ---------- | ---------------- | ------------------------- |
| Serial0 | OTG1 (USB) | MAVLink 2        | USB                       |
| Serial1 | USART1     | RC input         | RC                        |
| Serial2 | USART2     | MAVLink 2        | TELEM1                    |
| Serial3 | USART3     | None             | SERIAL3                   |
| Serial4 | UART7      | GPS              | GPS                       |
| Serial5 | UART4      | None             | SERIAL4                   |
| Serial6 | UART5      | None             | SERIAL5                   |
| Serial7 | USART6     | None             | SERIAL6                   |
| Serial8 | UART8      | Debug console    | DEBUG                     |
| Serial9 | OTG2 (USB) | SLCAN            | USB (shared with Serial0) |

Serial9 (OTG2) is permanently assigned as SLCAN and shares the USB connector with Serial0. This allows CAN bus inspection via a ground station connected over USB.

To change a serial port protocol, set the corresponding `SERIALn_PROTOCOL` parameter. Common values: `1` = MAVLink 1, `2` = MAVLink 2, `5` = GPS, `8` = Serial (passthrough), `23` = RCIN.

***

### CAN bus

The board has two independent CAN buses. Both use the STM32H743 FDCAN peripheral and support DroneCAN (formerly UAVCAN v1). CAN termination resistors (120 Ω) are fitted on the default connector layer.

Recommended ArduPilot configuration for DroneCAN on both buses:

```
CAN_P1_DRIVER = 1
CAN_D1_PROTOCOL = 1
CAN_P2_DRIVER = 2
CAN_D2_PROTOCOL = 1
```

For SLCAN inspection of CAN1 via USB (Serial9):

```
CAN_SLCAN_CPORT = 1    # Monitor CAN1
```

***

### Ethernet

The board includes a LAN8742A 100BASE-TX Ethernet PHY. The interface is enabled automatically at boot with no configuration required.

To use MAVLink 2 over UDP:

```
SERIAL2_PROTOCOL = 2      # or any free serial
# Then configure network via:
NET_ENABLE = 1
NET_IPADDR0..3 = 192.168.1.2   # static IP
NET_GWADDR0..3 = 192.168.1.1
NET_NETMASK0..3 = 255.255.255.0
NET_P1_TYPE = 1           # UDP client
NET_P1_IP0..3 = 192.168.1.100  # GCS IP
NET_P1_PORT = 14550
```

Typical use cases include a companion computer (Raspberry Pi, NVIDIA Orin) running ROS 2 / MAVROS connected via Ethernet for high-bandwidth telemetry and onboard processing.

***

### Analog inputs (ADC7, ADC8)

Two general-purpose analog inputs are available on J2 of the main connector stack (not broken out on the default connector layer — available on custom carrier boards).

| Signal | J2 pin | Input range | Notes                      |
| ------ | ------ | ----------- | -------------------------- |
| ADC7   | 67     | 0–3.3 V     | RC-filtered, ESD protected |
| ADC8   | 57     | 0–3.3 V     | RC-filtered, ESD protected |

These can be used for battery voltage sensing (via external resistor divider), airspeed sensors, or other analog sources. They are not pre-configured in the default hwdef and require parameter setup:

***

### IMU heater

An internal resistive heater maintains IMU temperature at 45 °C in cold environments, controlled automatically by ArduPilot. No user configuration is required.&#x20;

***

### Internal sensors

The following sensors are fitted internally and require no external connections:

| Sensor         | Interface   | Instance     | Notes                       |
| -------------- | ----------- | ------------ | --------------------------- |
| IIM-42652 (×2) | SPI3, SPI4  | IMU1, IMU2   | Primary and secondary IMU   |
| BMI055         | SPI2        | IMU3         | Tertiary IMU (accel + gyro) |
| MS5611 (×2)    | SPI3, SPI4  | BARO1, BARO2 | Dual barometers             |
| BMM350         | I2C3 (0x14) | COMPASS1     | Internal magnetometer       |

The internal compass is a fallback only. For accurate heading, use an external compass mounted away from power cables and motors, connected via the GPS port or I²C connector.
