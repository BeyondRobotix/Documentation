---
description: DroneCAN enabled AUAV absolute and differential pressure sensor
---

# Air Data Module

## Introduction

The Beyond Robotix Air Data Module makes use of differential and absolute pressure sensors to estimate airspeed and altitude. The module works over DroneCAN with a variety of autopilots.

<figure><img src="../.gitbook/assets/Iso-Photoroom.jpg" alt="" width="375"><figcaption></figcaption></figure>

There are 5 variants of the air data module, each with their own maximum speeds:

* L05D -> 45m/s
* L10D -> 63m/s
* L30D -> 111m/s
* L60D -> 149m/s
* L100D -> 193m/s

It's best to choose the lowest speed variant which your platform will not exceed the maximum speed of.



It can be purchased here:

{% embed url="https://www.beyondrobotix.com/products/air-data-module" %}



## Pinout

<table><thead><tr><th width="316" align="right">Pin</th><th>Function</th><th data-hidden>Notes</th></tr></thead><tbody><tr><td align="right">1</td><td>5V</td><td>Recommended: 4.5 - 6V</td></tr><tr><td align="right">2</td><td>CAN H</td><td></td></tr><tr><td align="right">3</td><td>CAN L</td><td></td></tr><tr><td align="right">4</td><td>GND</td><td></td></tr></tbody></table>

{% hint style="info" %}
The CAN ports are common that can be used to daisy chain to other CAN nodes.
{% endhint %}

The computation on the Air Data Module is performed on the [CAN Node](../can-node-system/micro-node.md). If you would like to terminate the CAN bus at the Air Data Module, a solder jumper labeled `120R` can be jumped.

<figure><img src="../.gitbook/assets/Air Data Module Disassembled.jpg" alt=""><figcaption></figcaption></figure>

## Parameters

#### NODEID: CAN Node ID&#x20;

ID of the CAN Node on the CAN bus

Range: 1-125

**SENSOR\_PRESSURE: Pressure Range of AUAV Sensor**

This is the type of AUAV sensor installed on the Air Data Module.&#x20;



L05D = 5

L10D = 10

L30D = 30

L60D = 60

L100D = 100



## Firmware

Our air data module runs on custom firmware which can be found here, including the latest firmware binaries:

{% embed url="https://github.com/BeyondRobotix/AUAV-DroneCAN/releases/latest" %}

To update to the latest firmware, there are two methods you can use:

* Mission Planner
* DroneCAN GUI&#x20;

### Updating: Mission Planner

1. Connect to your Flight Controller in Mission Planner
2. Go to Setup > Optional Hardware > UAVCAN
3. Click `CAN1` or `CAN2` (Depending on the bus you are connected to)
4. Click on `Menu` next to the Air Data Module
5. Click Update Firmware
6. Select the `.bin` file from the latest release page above

### Updating: DroneCAN GUI

1. Connect to your Flight Controller in DroneCAN GUI
2. Double Click on the Air Data Module
3. Click Update Firmware
4. Select the `.bin` file from the latest release page above

You are now on the latest version of the firmware!

## Mechanical

Images of hole spacing and overall dimensions are provided below. Additionally, the case STL files can also be found below.

### Hole spacing

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### Overall dimensions

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>





### PCB dimensions

Bounding box: 20x26x10.5mm

Please note: this product is designed to require the case. Without it, the Micro CAN node will need mechanically stabilising through another means.



### Case Download

You can 3D print the cases using the attached files:

{% file src="../.gitbook/assets/Airspeed Base (1).stl" %}

{% file src="../.gitbook/assets/Airspeed Lid (1).stl" %}
