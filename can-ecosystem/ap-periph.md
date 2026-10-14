---
description: Contains information relevant to using AP periph on the micro node system
icon: robot-astromech
---

# AP Periph

We support AP\_Periph based firmware's for our CAN nodes. Each board has its own AP\_Periph build, documented on that board's page:

* [Micro Node and CAN Node](l431-can-node.md#firmware) (STM32L431)
* [CAN Node Plus](can-node-plus.md#firmware) (STM32H723)

## Micro Node and CAN Node

The L431 nodes ship with a generic AP\_periph firmware by default.

### Default firmware features

* 6 PWMs/GPIOs
  * PA0 - PWM 1 - GPIO 50
  * PA1 - PWM 2 - GPIO 51
  * PA8 - PWM 3 - GPIO 52
  * PA9 - PWM 4 - GPIO 53
  * PA10 - PWM 5 - GPIO 54
  * PA11 - PWM 6 - GPIO 55
* Serial UBLOX/NMEA/SBF GPS (Serial 3 by default)
* Barometers
* Airspeed
* Battery (any, but ADC PA0 voltage and PA1 Current enabled by default)
* Relay



### Firmware files and flashing

If there is no AP periph bootloader present on the node (you've been using Arduino DroneCAN for example) then AP\_Periph\_with\_bl.hex needs flashing with STM cube Programmer using STLINK or UART.&#x20;

If the AP periph bootloader is present, the .bin file can be used in the usual way through mission planner.

{% hint style="info" %}
We've working on merging our hwdef file into Ardupilot repository
{% endhint %}

{% file src="../.gitbook/assets/AP_Periph.bin" %}

{% file src="../.gitbook/assets/ap-periph-firmware-with-bootloader.hex" %}

## CAN Node Plus

The CAN Node Plus runs AP\_Periph from our own bootloader rather than an AP\_Periph bootloader, so the same board can move between AP\_Periph and Arduino DroneCAN over CAN. Only the `.bin` file is needed. Defaults, port assignments and firmware files are on the board page:

{% content-ref url="can-node-plus.md" %}
[can-node-plus.md](can-node-plus.md)
{% endcontent-ref %}
