---
description: DroneCAN enabled AUAV absolute and differential pressure sensor
---

# AUAV Air Data Module

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

| Parameter        | Description                                                       | Accepted Values                                                       |
| ---------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------- |
| NodeID           | The ID the node will appear as in the CAN bus                     | 1-127                                                                 |
| Sensor\_Pressure | This is the type of AUAV sensor installed on the Air Data Module. | <p>L05D = 5<br>L10D = 10<br>L30D = 30<br>L60D = 60<br>L100D = 100</p> |

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

Images of hole spacing and overall dimensions are provided below. Additionally, the case STL files can&#x20;

### Case Specifications

<figure><img src="../.gitbook/assets/SLS Case.jpg" alt="" width="375"><figcaption></figcaption></figure>

Specifcations for the size and shape of the case:

{% file src="../.gitbook/assets/Mechanical Drawing Air Data Module Case.pdf" %}

Files to 3D print/edit the included case:

{% file src="../.gitbook/assets/BR AUAV ADM Lid.3mf" %}

{% file src="../.gitbook/assets/BR AUAV ADM Base.3mf" %}

### PCB Specifications

{% file src="../.gitbook/assets/Mechanical Drawing Air Data Module.pdf" %}

{% file src="../.gitbook/assets/AUAV DroneCAN Air Data Module.step" %}

Please note: this product is designed to require the case. Without it, the Micro CAN node will need mechanically stabilising through another means.

