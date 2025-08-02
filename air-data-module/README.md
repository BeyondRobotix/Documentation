---
description: DroneCAN enabled AUAV absolute and differential pressure sensor
---

# Air Data Module

## Intro

The Beyond Robotix Air Data Module makes use of differential and absolute pressure sensors to estimate airspeed and altitude. The module works over DroneCAN with a variety of autopilots.

<figure><img src="../.gitbook/assets/Iso-Photoroom.jpg" alt="" width="375"><figcaption></figcaption></figure>

There are three variands of the Air Data Module depending on airspeed.&#x20;

* Low speed: Up to 45m/s (88 kts)
* Medium speed: Up to 63 m/s (122 kts)
* High Speed: Up to 111 m/s (216 kts)



## Software

Our air data module runs on custom firmware which can be found here, including the latest firmware binaries:

{% embed url="https://github.com/BeyondRobotix/AUAV-DroneCAN" %}

It's written using our Arduino DroneCAN library.

