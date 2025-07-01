---
description: Set up your Kahuna in 3 easy steps
icon: diagram-project
---

# Quick Start Guide

**Step 1:**

Connect the Kahuna to your autopilot

**Step 2:**

Power on your GCS and your UAV

**Step 3:**

Connect your GCS Wi-Fi to the Kahuna \
&#xNAN;_&#x53;SID: BeyondRobotix_\
_Password: beyondrobotix_\
&#xNAN;_(Remember - the password has no capitals)_

Misson Planner or QGroundControl will connect to your UAV automatically!

***

If that didn't work, try setting these parameters in your autopilot.

If using **PX4**, set the following parameters:\
SER\_TEL1\_BAUD = 57600 (57600 8N1)

If using **ArduPilot**, set the following parameter\
SERIAL1\_BAUD = 57 (57600) \
SERIAL1\_PROTOCOL = 2 (MAVLink2)

{% hint style="info" %}
This assumes you are plugging the Kahuna into Telem1. If you are plugging it into Telem 2, use SERIAL2\_BAUD and SERIAL2\_PROTOCOL for Ardupilot and SER\_TEL2\_BAUD for PX4
{% endhint %}

{% hint style="warning" %}
Make sure to reboot your autopilot after changing parameters.
{% endhint %}

