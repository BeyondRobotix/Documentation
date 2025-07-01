---
description: Contains information relevant to using AP periph on the micro node system
icon: robot-astromech
---

# AP Periph

We support AP\_Periph based firmware's for our CAN node. The Nodes ship with a generic AP\_periph firmware by default.

## Default firmware features

* 4 PWMs/GPIOs
  * PA8 - PWM 1 - GPIO 50
  * PA9 - PWM 2 - GPIO 51
  * PA10 - PWM 3 - GPIO 52
  * PA11 - PWM 4 - GPIO 53
* Serial UBLOX/NMEA GPS (Serial 3 by default)
* Barometers
* Airspeed
* Battery (any, but ADC PA0 voltage and PA1 Current enabled by default)
* Relay



## Firmware files and flashing

If there is no AP periph bootloader present on the node (you've been using Arduino DroneCAN for example) then AP\_Periph\_with\_bl.hex needs flashing with STM cube Programmer using STLINK or UART.&#x20;

If the AP periph bootloader is present, the .bin file can be used in the usual way through mission planner.

{% hint style="info" %}
We've working on merging our hwdef file into Ardupilot repository
{% endhint %}

{% file src="../.gitbook/assets/AP_Periph_with_bl.hex" %}
For use with STM Cube Programmer
{% endfile %}

{% file src="../.gitbook/assets/AP_Periph (3).bin" %}
If the AP periph booloader is present on the node, this file can be used to program the firmware through mission planner.
{% endfile %}
