# Default Carrier Layer Pinout

The carrier board pinouts follow Pixhawk standard pinouts! 


#### RC (Serial1 - USART1)

6-pin, JST-GH 1.25 mm pitch.

Default protocol: RC input (SBUS/CRSF/etc.)\
ArduPilot parameter: `SERIAL1_PROTOCOL`

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX             | Output    | 3.3 V   |
| 3   | RX             | Input     | 3.3 V   |
| 4   | NC             | -         | -       |
| 5   | NC             | -         | -       |
| 6   | GND            | -         | 0 V     |

***

#### TELEM1 (Serial2 - USART2)

6-pin, JST-GH 1.25 mm pitch.

Default protocol: MAVLink 2\
ArduPilot parameter: `SERIAL2_PROTOCOL`\
Power rail: Telem 5 V (current-limited, see [Power](#power))

| Pin | Signal     | Direction | Voltage |
| --- | ---------- | --------- | ------- |
| 1   | Telem 1 5V | Output    | 5 V     |
| 2   | TX         | Output    | 3.3 V   |
| 3   | RX         | Input     | 3.3 V   |
| 4   | CTS        | Input     | 3.3 V   |
| 5   | RTS        | Output    | 3.3 V   |
| 6   | GND        | -         | 0 V     |

> This is the primary telemetry port. The 5 V rail on this connector is sourced from a dedicated current-limited supply separate from the general peripherals rail.

***

#### SERIAL3 (USART3)

6-pin, JST-GH 1.25 mm pitch.

Default protocol: None (user-defined)\
ArduPilot parameter: `SERIAL3_PROTOCOL`

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX             | Output    | 3.3 V   |
| 3   | RX             | Input     | 3.3 V   |
| 4   | CTS            | Input     | 3.3 V   |
| 5   | RTS            | Output    | 3.3 V   |
| 6   | GND            | -         | 0 V     |

***

#### SERIAL4 / SERIAL5 / SERIAL6

6-pin, JST-GH 1.25 mm pitch.

Default protocol: None (user-defined)

| ArduPilot port | MCU peripheral | Parameter          |
| -------------- | --------------- | ------------------ |
| Serial4        | UART4           | `SERIAL4_PROTOCOL` |
| Serial5        | UART5           | `SERIAL5_PROTOCOL` |
| Serial6        | USART6          | `SERIAL6_PROTOCOL` |

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX             | Output    | 3.3 V   |
| 3   | RX             | Input     | 3.3 V   |
| 4   | NC             | -         | -       |
| 5   | NC             | -         | -       |
| 6   | GND            | -         | 0 V     |

***

#### GPS (Serial7 - UART7)

6-pin, JST-GH 1.25 mm pitch.

Default protocol: GPS\
ArduPilot parameter: `SERIAL7_PROTOCOL`\
This port also carries I²C for an external compass.

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | TX7            | Output    | 3.3 V   |
| 3   | RX7            | Input     | 3.3 V   |
| 4   | SCL            | Output    | 3.3 V   |
| 5   | SDA            | Output    | 3.3 V   |
| 6   | GND            | -         | 0 V     |

> This is the recommended GPS port.

***

#### CAN1 / CAN2

4-pin, JST-GH 1.25 mm pitch. Termination: 120 Ω installed on the connector layer.

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | CAN*n*\_H      | Bidir     | -       |
| 3   | CAN*n*\_L      | Bidir     | -       |
| 4   | GND            | -         | 0 V     |

> *n* is 1 or 2 depending on the connector. CAN1 maps to FDCAN1 (`CAN_P1_DRIVER = 1`), CAN2 maps to FDCAN2 (`CAN_P2_DRIVER = 1`).

***

#### I²C

4-pin, JST-GH 1.25 mm pitch.

| Pin | Signal         | Direction | Voltage |
| --- | -------------- | --------- | ------- |
| 1   | Peripherals 5V | Output    | 5 V     |
| 2   | SCL            | Output    | 3.3 V   |
| 3   | SDA            | Output    | 3.3 V   |
| 4   | GND            | -         | 0 V     |

> I²C bus 2. Suitable for external compasses, barometers, or airspeed sensors that require I²C. The bus is buffered on the power layer.

***

#### ETH (Ethernet)

4-pin, JST-GH 1.25 mm pitch.

| Pin | Signal | Direction | Voltage |
| --- | ------ | --------- | ------- |
| 1   | TX\_P  | Output    | -       |
| 2   | TX\_N  | Output    | -       |
| 3   | RX\_P  | Input     | -       |
| 4   | RX\_N  | Input     | -       |

***

#### USB

4-pin, JST-GH 1.25 mm pitch.

| Pin | Signal        | Direction | Voltage |
| --- | ------------- | --------- | ------- |
| 1   | Pwr\_USB2\_In | Input     | 5 V     |
| 2   | USB\_P        | Bidir     | -       |
| 3   | USB\_N        | Bidir     | -       |
| 4   | GND           | -         | 0 V     |


***

#### DEBUG

14-pin, 1.27 mm pitch (FTSH-107-01-L-DV-K, compatible with standard ARM SWD cables).

| Pin | Signal |
| --- | ------ |
| 1   |        |
| 2   |        |
| 3   | 3v3    |
| 4   | SWDIO  |
| 5   | GND    |
| 6   | SWCLK  |
| 7   | GND    |
| 8   |        |
| 9   |        |
| 10  |        |
| 11  | GND    |
| 12  |        |
| 13  | RX8    |
| 14  | TX8    |

> UART8 is the ArduPilot debug console (`STDOUT_SERIAL SD8`, 57600 baud). Connect a 3.3 V FTDI cable to TX8/RX8 to view boot output and ArduPilot log messages.

***

#### PWR1 / PWR2

6-pin, Molex CLIK-Mate 2.0 mm pitch power input connectors. PWR1 and PWR2 provide independent redundant power paths - either connector alone is sufficient to power the board.

| Pin | Signal  | Direction | Voltage    |
| --- | ------- | --------- | ---------- |
| 1   | Pwr\_In | Input     | 4.2–5.75 V |
| 2   | Pwr\_In | Input     | 4.2–5.75 V |
| 3   | CAN2\_H | Bidir     | -          |
| 4   | CAN2\_L | Bidir     | -          |
| 5   | GND     | -         | 0 V        |
| 6   | GND     | -         | 0 V        |

> The power connectors CAN2 for our power input module with current/voltage monitoring

***

#### PWM outputs

The 16 PWM outputs are available on a 3×16 row of 2.54 mm pitch through-holes at the bottom edge of the board, with three rows labelled **PWM**, **Spwr** (servo power), and **GND**.

| Row  | Function                                                                                                    |
| ---- | ----------------------------------------------------------------------------------------------------------- |
| PWM  | Signal (level-shifted, 3.3 V or 5 V - see below)                                                            |
| Spwr | Servo rail power (**not a power output** - requires external regulator) |
| GND  | Ground                                                                                                      |

PWM outputs 1–8 are the **MAIN** outputs and pass through the level shifter (voltage-selectable). PWM outputs 9–16 are **AUX** outputs and are direct 3.3 V GPIO.

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
