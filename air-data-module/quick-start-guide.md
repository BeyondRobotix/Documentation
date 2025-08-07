# Quick Start Guide

The Beyond Robotix Air Data Module includes the following:

* AUAV Sensor Board
* Micro CAN Node
* Pitot Static Tube
* Silicone tube
* Case

<figure><img src="../.gitbook/assets/ADM_bundle_edited-Photoroom.jpg" alt="" width="375"><figcaption></figcaption></figure>

## Assembly

1. Feed the tube through the lid of the case until there is about 1 mm protruding through the lid.&#x20;

<div><figure><img src="../.gitbook/assets/IMG_1193 (1).JPG" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/IMG_1194 (1).JPG" alt=""><figcaption></figcaption></figure></div>

2. Insert the AUAV sensor into the tubes and pull the tubes gently until the sensor is flush with the plastic

<div><figure><img src="../.gitbook/assets/IMG_1195.JPG" alt=""><figcaption><p>Before pulling flush</p></figcaption></figure> <figure><img src="../.gitbook/assets/IMG_1197.JPG" alt=""><figcaption><p>After pulling flush</p></figcaption></figure> <figure><img src="../.gitbook/assets/IMG_1198.JPG" alt=""><figcaption><p>Top View</p></figcaption></figure></div>

3. Slide the sensor assembly into the main case and press until you feel a click

<div><figure><img src="../.gitbook/assets/IMG_1199 (1).JPG" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/IMG_1202 (1).JPG" alt=""><figcaption></figcaption></figure></div>

4. Fix the pitot static tube to the tubes from the airspeed sensor.&#x20;

<figure><img src="../.gitbook/assets/Hoses.png" alt="" width="375"><figcaption></figcaption></figure>

{% hint style="success" %}
Ensure the **high** side (H) of the sensor is connected to the **straight** barb on the pitot-static tube (going to the total pressure port) and the **low** side (L) of the sensor to the **diagonal** barb (going to the static ports).
{% endhint %}

5. Connect to one of your CAN ports on your flight controller

<figure><img src="../.gitbook/assets/Connected.png" alt="" width="375"><figcaption></figcaption></figure>

5. Set the following parameters, if they are not already set:
   1. CAN\_D1\_PROTOCOL = 1 (DroneCAN)
   2. CAN\_P1\_DRIVER = 1 (First Driver)
   3. ARSPD\_ENABLE = 1 (ENABLE)
   4. ARSPD\_TYPE = 8 (DroneCAN)
6. Save parameters and reboot your autopilot. You should be able to see the airspeed and barometer sensor in the HW ID tab of Mission Planner.

<figure><img src="../.gitbook/assets/MP.png" alt=""><figcaption><p>Airspeed and barometer show up as UAVCAN</p></figcaption></figure>

