---
description: Rappidly integrate a compact CAN enabled STM32 processor
icon: industry
---

# Micro Node

<figure><img src="../.gitbook/assets/Front and Back.png" alt="" width="375"><figcaption><p>Micro Node</p></figcaption></figure>

The Beyond Robotix Micro Node allows you to create production grade PCBs without needing to worry about the complicated bit. Put the board to board connector on your project, wire in a sensor, a connector and now you have a CAN enabled project.

We do Quality Assurance and software upload, meaning you just have to screw the board down to your project.

## Store

{% embed url="https://www.beyondrobotix.com/products/micro-can-node" %}

{% embed url="https://www.beyondrobotix.com/products/micro-can-node-development-bundle" %}

## Specifications

* Dimensions: 26 x 20 mm
* Mass: 2.2 g
* Power Consumption: 30 mA (40 mA peak) at 5V
* STM32L431CCU6 Processor with 256 KB of flash
* 200mA LDO
* Short Circuit Protection - 1 A Input Fuse
* Reverse Polarity Protection - Input Diode
* Optional 120 ohm termination resistor via solder pad
* Status LED
* High reliability connector & mechanical fixing
* Screw heads accessible from the top of the board
* AP periph firmware support
* Arduino DroneCAN support

## Pinout

The Micro Node requires the **DF40C-80DS-0.4V(51)** connector to be on the carrier board.&#x20;

{% embed url="https://www.mouser.co.uk/ProductDetail/Hirose-Connector/DF40C-80DS-0.4V51?qs=eDUdFcBPps3TI2tU7fRYSg%3D%3D" %}
CAN node carrier board connector
{% endembed %}

{% hint style="info" %}
The equivalent JLCPCB component is C312960
{% endhint %}

The footprint can be downloaded from the mouser page. Be sure to use this one! Pin numbers can be different on different footprints for the same part. We learnt the hard way.

<figure><img src="../.gitbook/assets/Screenshot 2025-03-08 212952.png" alt=""><figcaption><p>Footprint pinout mapping. 1 bottom right, 2 top right.</p></figcaption></figure>

You'll notice our connector has far many more pins than the STM32 has outputs, we've chosen this due to stock availability, as well as we'll be able to use this same connector on future projects on STMs with higher pin counts.

| Pin Number | AP\_Periph function                           | STM/Physical                                   |
| ---------- | --------------------------------------------- | ---------------------------------------------- |
| 1          |                                               | GND                                            |
| 2          |                                               | <mark style="background-color:red;">GND</mark> |
| 6          |                                               | +3.3V                                          |
| 7          |                                               | CAN\_H                                         |
| 9          |                                               | CAN\_L                                         |
| 15         |                                               | +5V                                            |
| 16         |                                               | +5V                                            |
| 32         |                                               | PB12                                           |
| 33         | <mark style="color:blue;">USART3\_RX</mark>   | PB11                                           |
| 34         | <mark style="color:purple;">SCL</mark>        | PB13                                           |
| 35         | <mark style="color:blue;">USART3\_TX</mark>   | PB10                                           |
| 36         | <mark style="color:purple;">SDA</mark>        | PB14                                           |
| 37         | <mark style="color:orange;">SPI1\_CS2</mark>  | PB2                                            |
| 38         |                                               | PB15                                           |
| 39         |                                               | PB1                                            |
| 40         |                                               | PA8                                            |
| 41         |                                               | PB0                                            |
| 42         |                                               | PA9                                            |
| 43         | <mark style="color:orange;">SPI1\_MOSI</mark> | PA7                                            |
| 44         |                                               | PA10                                           |
| 45         |                                               | PA6                                            |
| 46         |                                               | PA11                                           |
| 47         | <mark style="color:orange;">SPI\_SCK</mark>   | PA5                                            |
| 48         |                                               | PA11                                           |
| 49         | <mark style="color:orange;">SPI\_CS1</mark>   | PA4                                            |
| 50         | <mark style="color:red;">SWD</mark>           | PA13                                           |
| 51         | <mark style="color:blue;">USART2\_RX</mark>   | PA3                                            |
| 53         | <mark style="color:blue;">USART2\_TX</mark>   | PA2                                            |
| 55         |                                               | PA1                                            |
| 57         |                                               | PA0                                            |
| 59         |                                               | NRST                                           |
| 66         | <mark style="color:red;">SWC</mark>           | PA14                                           |
| 68         |                                               | PA15                                           |
| 70         | <mark style="color:orange;">SPI1\_MISO</mark> | PB4                                            |
| 72         |                                               | PB5                                            |
| 74         | <mark style="color:blue;">USART1\_TX</mark>   | PB6                                            |
| 76         | <mark style="color:blue;">USART1\_RX</mark>   | PB7                                            |
| 78         |                                               | BOOT0                                          |
| 79         |                                               | GND                                            |
| 80         |                                               | PC13                                           |

## Standoffs

The design requires 4x M1.6, 1.5mm standoffs. An example of these is the 9774015633R from Wurth Electronik.

{% embed url="https://www.mouser.co.uk/ProductDetail/Wurth-Elektronik/9774015633R?qs=2WXlatMagcGvxYPtvinz1A%3D%3D" %}

{% hint style="info" %}
The equivalent JLCPCB component is C2928168
{% endhint %}

## Connector position drawing

This drawing is for the hole and connector positions for the **carrier board,** not the node itself.

<div align="left"><figure><img src="../.gitbook/assets/Node Development Board Layout (1).png" alt="" width="375"><figcaption><p>Layout with Dimensions</p></figcaption></figure> <figure><img src="../.gitbook/assets/Node Development Board Layout Real.png" alt="" width="304"><figcaption><p>Example layout on Node Development Board V1.0</p></figcaption></figure></div>

## Template

A Kicad template file with required libraries can be used as a starting point to make a custom carrier board.

<figure><img src="../.gitbook/assets/Node Development Board Layout Render.png" alt=""><figcaption></figcaption></figure>

{% file src="../.gitbook/assets/Micro Node Carrier Template Rev A.zip" %}

## Software

**Arduino DroneCAN**

We wrote an Arduino library for using DroneCAN on this Node out of the box!

You can integrate your sensor how you like with minimal code. Includes bootloader support so your apps can be updated over CAN.

{% content-ref url="arduino-dronecan/" %}
[arduino-dronecan](arduino-dronecan/)
{% endcontent-ref %}

**AP\_Periph**

We support AP\_Periph based firmware's for our CAN node. The Nodes ship with a generic AP\_periph firmware by default.

{% content-ref url="ap-periph.md" %}
[ap-periph.md](ap-periph.md)
{% endcontent-ref %}

### Custom Firmware

We can flash a custom/specific firmware for your order on your request. Contact us for custom firmware to be written at _**admin@beyondrobotix.com**_

We have experience integrating virtually every type of sensor into communicating with Ardupilot or PX4, for example fuel flow sensors, ultrasound sensors, GNSS receivers, magnetometers, pressure transducers,  power monitors and many more. It is also possible to adapt the output of the Micro Node to send PWM, GPIO, CAN, Serial, I2C and SPI.

## Environmental

Based on environmental ratings for the components and board manufacturer statements. Not a tested figure by Beyond Robotix.

The boards can be conformally coated on request. If environmental considerations are a requirement for your project, contact us at _**admin@beyondrobotix.com**_ for further insight and options.

* Temperature > -20 °C to 85 °C ambient temperature

## QA Testing

We test every board we dispatch. Our process is:

* Mount the node to a test carrier
* Upload testing firmware
* Our software checks every pin is functioning as expected
* Upload customer firmware
* Check node status can be seen through CAN interface (if applicable)
* Package for dispatch
