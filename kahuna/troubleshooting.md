---
description: Common issues and how to fix them
icon: circle-exclamation
---

# Troubleshooting

#### Air (red) LED blinking when Kahuna is plugged into the autopilot?

The Kahuna is not receiving a heartbeat from your autopilot. Ensure that the baud rate of the serial port which the Kahuna is connected to matches the baud rate of the Kahuna, by default the Kahuna baud rate is 57600.

#### Connected to Wi-Fi but not to ground control station software (solid green LED, blinking blue LED)?

This is most likely caused by your firewall. You need to ensure that your GCS software is allowed to receive incoming connections on port 14550. This can be done by allowing your GCS software through the firewall.

1. Search for "Firewall & network protection" in the start menu search bar
2. Select "Allow an app through the firewall" and find your GCS software in the list
3. Click on "Change settings" and then make sure the box for "Private" is checked
4. If your GCS software is not in the list, click on "Allow another app..." and then in the menu that opens click on "Browse..." and then find and select the .exe of your GCS software. For Mission Planner this is usually `C:\Program Files (x86)\Mission Planner\MissionPlanner.exe`. For QGroundControl this is usually `C:\Program Files\QGroundControl\QGroundControl.exe`.&#x20;

Now that your GCS is allows to communicate on private networks, you need to set up the Kahuna Wi-Fi network as a private network.

1. Select Start , then type "settings". Select Settings > Network & internet > Wi-Fi.
2. On the Wi-Fi settings screen, select Manage known networks, and select the Kahuna Wi-Fi network you're connected to.
3. On the Wi-Fi network screen, under Network profile type, select Public (Recommended)







