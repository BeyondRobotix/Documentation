---
description: DroneCAN enabled AUAV absolute and differential pressure sensor
---

# Air Data Module

## Introduction

The Beyond Robotix Air Data Modules makes use of the Allsensors AUAV sensor. This module has a differential and absolute pressure sensors to estimate airspeed and altitude. The module works over DroneCAN with a variety of autopilots.

<figure><img src="../.gitbook/assets/Modules.png" alt=""><figcaption></figcaption></figure>

There are 5 variants of the air data module, each with their own maximum speeds:

* L05D → 45m/s&#x20;
* L10D → 63m/s
* L30D → 111m/s
* L60D → 149m/s
* L100D → 193m/s

For the best performance, choose the variant with the lowest speed that your platform will not exceed the maximum speed of. For example, if your platforms maximum speed is 50m/s, choose the L10D with the upper limit of 63 m/s. The L05D is not suitable as you will saturate the sensor. Going marginally over the maximum speed of the sensor will not damage it. The proof pressure (the point where the sensor will exhibit permenant damage) is 67 kPa, which equates to about 330 m/s - approximately Mach 0.97!

### Buy Now

{% embed url="https://www.beyondrobotix.com/products/air-data-module" %}
