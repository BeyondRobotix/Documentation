---
icon: gear
---

# Advanced Configuration


## Power monitoring

Four current sense channels are available for ArduPilot battery monitor instances:

| ArduPilot instance | MCU pin     | Sense point             | hwdef name           |
| ------------------ | ----------- | ----------------------- | -------------------- |
| BATT5              | PC3 (ADC1)  | PWR1 input current      | `BATT5_CURRENT_SENS` |
| BATT6              | PF11 (ADC1) | PWR2 input current      | `BATT6_CURRENT_SENS` |
| BATT7              | PA6 (ADC1)  | USB power layer current | `BATT7_CURRENT_SENS` |
| BATT8              | PF13 (ADC2) | USB carrier current     | `BATT8_CURRENT_SENS` |

These measure current only. The board does not have direct access to raw battery voltage — voltage monitoring should be done externally via a DroneCAN power module, or via a resistor divider on ADC7 or ADC8 (J2 pins 67 and 57).

A 5 V bus sense line (`VDD_5V_SENS` on PF3, ADC3) monitors the internal 5 V rail through a 1:1 divider and is configured with `SCALE(2)`.

***

## PWM voltage selection

PWM outputs 1–8 (MAIN) pass through level-shifting buffers whose output voltage is software-selectable between 3.3 V and 5 V. The default in the firmware is **3.3 V**.

To switch to 5 V output (required for most standard RC servos):

```
SERVO_BLH_POLES = 0    # not related, just context
# Set GPIO 3 HIGH for 5V, LOW for 3.3V
```

In ArduPilot, control this via the `HAL_GPIO_PWM_VOLT_PIN` (GPIO 3, PI6):

```python
# From GCS or scripting:
param set BRD_PWM_VOLT_EN 1     # if available in your build
```

Or directly via the `relay` interface:

```
RELAY1_PIN = 3        # GPIO 3 = PWM voltage selector
RELAY1_DEFAULT = 1    # 1 = HIGH = 5V output
```

Then use `do-set-relay` in a mission or MAVLink command to toggle. Set to HIGH (1) for 5 V servo output, LOW (0) for 3.3 V.

> PWM outputs 9–18 (AUX) are direct GPIO outputs and are always 3.3 V regardless of this setting.

***

## Relay / GPIO outputs

Two additional high-current enable outputs control the 5V power rails and can also be used as relays if power switching is needed:

| Function                         | MCU pin | GPIO number | Default   |
| -------------------------------- | ------- | ----------- | --------- |
| HI\_PWR\_EN (Telem 5V)           | PJ0     | 85          | HIGH (on) |
| PERIPH\_PWR\_EN (Peripherals 5V) | PJ1     | 86          | HIGH (on) |

> **Caution:** disabling PERIPH\_PWR\_EN (GPIO 86) will cut power to all peripherals on the Peripherals 5V rail. Only use this as a relay if you understand the downstream effects.

***

## Ethernet

The board includes a LAN8742A 100BASE-TX Ethernet PHY. The interface is enabled automatically at boot with no configuration required.

To use MAVLink 2 over UDP:

```
SERIAL2_PROTOCOL = 2      # or any free serial
# Then configure network via:
NET_ENABLE = 1
NET_IPADDR0..3 = 192.168.1.2   # static IP
NET_GWADDR0..3 = 192.168.1.1
NET_NETMASK0..3 = 255.255.255.0
NET_P1_TYPE = 1           # UDP client
NET_P1_IP0..3 = 192.168.1.100  # GCS IP
NET_P1_PORT = 14550
```

Typical use cases include a companion computer (Raspberry Pi, NVIDIA Orin) running ROS 2 / MAVROS connected via Ethernet for high-bandwidth telemetry and onboard processing.

