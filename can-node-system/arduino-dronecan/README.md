---
description: >-
  A library to send CAN messages compatible with Ardupilot and PX4 UAV
  autopilots.
icon: binary
---

# Arduino DroneCAN

## About

We made a library to make DroneCAN development as simple as possible. the Arduino DroneCAN repository allows you to get started with Ardupilot/PX4 compatible CAN messages and functionality straight out of the box using Beyond Robotix CAN node hardware.&#x20;

{% hint style="warning" %}
This documentation is written for the 1.3.X version of the library. If you're on a different version, the main.cpp file of your version should demonstrate the interface of that version.&#x20;
{% endhint %}

Github Repository:

{% embed url="https://github.com/BeyondRobotix/Arduino-DroneCAN" %}

## Installation

There are a few ways of working with Arduino Code, we recommend the following steps for seamless integration with the project.

1. Install Visual Studio Code [https://code.visualstudio.com/download](https://code.visualstudio.com/download)
2. Install the PlatformIO extension [https://platformio.org/install/ide?install=vscode](https://platformio.org/install/ide?install=vscode)
3. Download the project [https://github.com/BeyondRobotix/Arduino-DroneCAN/releases/tag/v1.3.3](https://github.com/BeyondRobotix/Arduino-DroneCAN/releases/tag/v1.3.3)
4. Connect your STLINK to your CAN node
5. Press upload!



If you want to use the master branch, remember to clone submodules:

```
git clone --recurse-submodules https://github.com/BeyondRobotix/Arduino-DroneCAN.git 
```

## Bootloader details

ArduinoDroneCAN uses a modified AP\_Bootloader to allow app updates over CAN. The standard AP\_Bootloader cannot be used. The bootloader bin can be found in the project repository.&#x20;

From release 1.3.1 the bootloader is flashed at the same time as the app flashing using the "Micro-Node-Bootloader" configuration.&#x20;



## Breakpoint debugging

We haven't been able to successfully start breakpoint debugging while a bootloader is on the node. So far.&#x20;

For now, the "Micro-Node-No-Bootloader" platformio configuration is required. Remember, you will no longer have the bootloader so you won't be able to update programs over CAN. Once you've done your debugging sessions, you can then switch back to the other config for app deployment!



## Detailed Example

The Arduino DroneCAN project reduces using DroneCAN down to creating an object, initialising the object, and one method to call in your loop function.&#x20;

### General advice

* Avoid the delay() function! this will disrupt any CAN activities. We show best practises for your loop function later
* Respect the watchdog. IWatchdog.reload() needs calling within the timeout time (2 seconds in our examples)
* Following the format of our example [https://github.com/BeyondRobotix/Arduino-DroneCAN/blob/master/src/main.cpp](https://github.com/BeyondRobotix/Arduino-DroneCAN/blob/master/src/main.cpp) will reduce the likelyhood of strange behaviour.
* If your program starts acting up, strip your program back to our example and build gradually from there. You'll find where bugs get introduced.
* If your node goes into "maintenance" mode then it means your program is crashing

### The Setup

app\_setup, dronecan.init and IWatchdog.begin must all be called at the start of setup. Doing another activities before or between functions may result in instabilities. Remember, the watchdog is running after this so call IWatchdog.reload() if you do any long running activities.&#x20;

#### app\_setup()

```cpp
app_setup();
```

app\_setup is required to be called at the start of the Arduino setup() function. This function is required as part of the transfer from the bootloader to app control. The function is internally disabled when non bootloader mode is selected in PlatformIO.

#### dronecan.init()

```cpp
dronecan.version_major = 1;
dronecan.version_minor = 0;
dronecan.init(
    onTransferReceived, 
    shouldAcceptTransfer, 
    custom_parameters,
    "Beyond Robotix Node"
);
```

Next, we initialise the dronecan object and optionally set version number of your software to appear in NodeInfo. In the init() method, we pass two functions which we'll explain later, a parameter list and lastly the name of the node for NodeInfo.

#### Watchdog

```cpp
IWatchdog.begin(2000000);
```

Our program requires starting the watchdog, remove it at your own peril! (it's started in the bootloader and from our testing, starting it here again is best)&#x20;



### DroneCAN looping

We can't use the loop() function normally used in Arduino, for some reasons related to bootloader things which we haven't solved yet.&#x20;

Instead, we do a while(true)



#### Fixed interval loops

The following code sets us up to run our if statement at 10Hz, checking if 100ms has passed since our last call. This is because we want to only send our CAN message at 10Hz and we want to call our "dronecan.cycle()" function to be called as much as possible to ensure CAN messages are send and received in a timely manner. We want to avoid delay() as much as possible!

```cpp
const uint32_t now = millis();

    // send our battery message at 10Hz
    if (now - looptime > 100)
    {
        looptime = millis();
```



#### Sensor reading

Next, we want to read in our sensor value. This could be from anything, a current monitor, position sensor.. in this example, we read in the temperature of our STM32 processor. You would have had to have initialised your sensor before the while(true) statement of course.

```cpp
int32_t vref = __LL_ADC_CALC_VREFANALOG_VOLTAGE(analogRead(AVREF), LL_ADC_RESOLUTION_12B);
int32_t cpu_temp = __LL_ADC_CALC_TEMPERATURE(vref, analogRead(ATEMP), LL_ADC_RESOLUTION_12B);
```



#### Sending DroneCAN messages

Next, we initialise our DroneCAN battery message packet and we assign one of its attributes a value from when we read in our sensor. We've used a battery message, since Ardupilot supports 8 of these by default, Mission planner can display information from any of these 8 and Ardupilot logs all battery instances. There may be messages suitable for your application, but be aware, Ardupilot does not support many DroneCAN messages.

```cpp
uavcan_equipment_power_BatteryInfo pkt {};
pkt.temperature = cpu_temp;
```



Finally,  we send our DroneCAN message. For some equipment type can messages, we can use this very simple function:

```cpp
sendUavcanMsg(dronecan.canard, pkt);
```



For messages which are not in this type list:

```python
headers = [
    "dronecan.sensors.hygrometer.Hygrometer.h",
    "dronecan.sensors.magnetometer.MagneticFieldStrengthHiRes.h",
    "dronecan.sensors.rc.RCInput.h",
    "dronecan.sensors.rpm.RPM.h",
    "uavcan.equipment.actuator.ArrayCommand.h",
    "uavcan.equipment.actuator.Command.h",
    "uavcan.equipment.actuator.Status.h",
    "uavcan.equipment.ahrs.MagneticFieldStrength.h",
    "uavcan.equipment.ahrs.MagneticFieldStrength2.h",
    "uavcan.equipment.ahrs.RawIMU.h",
    "uavcan.equipment.ahrs.Solution.h",
    "uavcan.equipment.air_data.AngleOfAttack.h",
    "uavcan.equipment.air_data.IndicatedAirspeed.h",
    "uavcan.equipment.air_data.RawAirData.h",
    "uavcan.equipment.air_data.Sideslip.h",
    "uavcan.equipment.air_data.StaticPressure.h",
    "uavcan.equipment.air_data.StaticTemperature.h",
    "uavcan.equipment.air_data.TrueAirspeed.h",
    "uavcan.equipment.camera_gimbal.AngularCommand.h",
    "uavcan.equipment.camera_gimbal.GEOPOICommand.h",
    "uavcan.equipment.camera_gimbal.Mode.h",
    "uavcan.equipment.camera_gimbal.Status.h",
    "uavcan.equipment.device.Temperature.h",
    "uavcan.equipment.esc.RawCommand.h",
    "uavcan.equipment.esc.RPMCommand.h",
    "uavcan.equipment.esc.Status.h",
    "uavcan.equipment.esc.StatusExtended.h",
    "uavcan.equipment.gnss.Auxiliary.h",
    "uavcan.equipment.gnss.ECEFPositionVelocity.h",
    "uavcan.equipment.gnss.Fix.h",
    "uavcan.equipment.gnss.Fix2.h",
    "uavcan.equipment.gnss.RTCMStream.h",
    "uavcan.equipment.hardpoint.Command.h",
    "uavcan.equipment.hardpoint.Status.h",
    "uavcan.equipment.ice.FuelTankStatus.h",
    "uavcan.equipment.ice.reciprocating.CylinderStatus.h",
    "uavcan.equipment.ice.reciprocating.Status.h",
    "uavcan.equipment.indication.BeepCommand.h",
    "uavcan.equipment.indication.LightsCommand.h",
    "uavcan.equipment.indication.RGB565.h",
    "uavcan.equipment.indication.SingleLightCommand.h",
    "uavcan.equipment.power.BatteryInfo.h",
    "uavcan.equipment.power.CircuitStatus.h",
    "uavcan.equipment.power.PrimaryPowerSupplyStatus.h",
    "uavcan.equipment.range_sensor.Measurement.h",
    "uavcan.equipment.safety.ArmingStatus.h",
]
```



we have to do the full boilerplate for a UAVCAN/DroneCAN message:

```cpp
uint8_t buffer[UAVCAN_EQUIPMENT_POWER_BATTERYINFO_MAX_SIZE];
uint32_t len = uavcan_equipment_power_BatteryInfo_encode(&pkt, buffer);
static uint8_t transfer_id;
canardBroadcast(&dronecan.canard,
        UAVCAN_EQUIPMENT_POWER_BATTERYINFO_SIGNATURE,
        UAVCAN_EQUIPMENT_POWER_BATTERYINFO_ID,
        &transfer_id,
        CANARD_TRANSFER_PRIORITY_LOW,
        buffer,
        len);
```



#### End of the loop

```cpp
dronecan.cycle();
IWatchdog.reload();
```

Outside our fixed interval if statment, we call our dronecan.cycle() method, which is required, and we also reset our watchdog timer. &#x20;



### Reading in CAN packets

onTransferReceived and shouldAcceptTransfer are required to be defined in our program and passed to dronecan.init. These functions are used to recieve any DroneCAN messages. We've put an example of decoding a Magnetometer message below. in onTransferRecieved, you can manipulate the recieved packet and use gobal variables or parameters etc to transfer information out of this function. DroneCANonTransferReceived must always be called at the end of this function for our library to work.

to allow a different message to be recieved, shouldAcceptTransfer needs the new message adding to it, following the same pattern as the magnetometer message example. As before, the last line of this function must remain the same for our library to work correctly.&#x20;

```cpp
/*
This function is called when we receive a CAN message, and it's accepted by the shouldAcceptTransfer function.
We need to do boiler plate code in here to handle parameter updates and so on, but you can also write code to interact with sent messages here.
*/
static void onTransferReceived(CanardInstance *ins, CanardRxTransfer *transfer)
{

    // switch on data type ID to pass to the right handler function
    // if (transfer->transfer_type == CanardTransferTypeRequest)
    // check if we want to handle a specific service request
    switch (transfer->data_type_id)
    {

    case UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_ID:
    {
        uavcan_equipment_ahrs_MagneticFieldStrength pkt{};
        uavcan_equipment_ahrs_MagneticFieldStrength_decode(transfer, &pkt);
        break;
    }
    }

    DroneCANonTransferReceived(dronecan, ins, transfer);
}

/*
For this function, we need to make sure any messages we want to receive follow the following format with
UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_ID as an example
 */
static bool shouldAcceptTransfer(const CanardInstance *ins,
                                 uint64_t *out_data_type_signature,
                                 uint16_t data_type_id,
                                 CanardTransferType transfer_type,
                                 uint8_t source_node_id)

{
    if (transfer_type == CanardTransferTypeBroadcast)
    {
        // Check if we want to handle a specific broadcast packet
        switch (data_type_id)
        {
        case UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_ID:
        {
            *out_data_type_signature = UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_SIGNATURE;
            return true;
        }
        }
    }

    return false || DroneCANshoudlAcceptTransfer(ins, out_data_type_signature, data_type_id, transfer_type, source_node_id);
}
```



### Other things

#### parameters

```cpp
std::vector<DroneCAN::parameter> custom_parameters = {
    { "NODEID", UAVCAN_PROTOCOL_PARAM_VALUE_INTEGER_VALUE, 69,  0, 127 },
    { "PARM_1", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_2", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_3", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
};
```

We passed the custom\_parameters object into dronecan.init. This parameters object needs "NODEID" in the list since we use that. The first value after the type dictates the default value, second the minimum for the parameter and the last the maximum. These min/maxes may or may not be shown in Mission planner etc and may or may not make any functional difference.



```cpp
dronecan.setParameter("PARM_1", 69);
Serial.print("PARM_1 value: ");
Serial.println(dronecan.getParameter("PARM_1"));
```

You can set and retrieve parameters easily. parameters get saved to flash so they'll be recalled on next boot.&#x20;



### Full example!

```cpp
#include <Arduino.h>
#include <dronecan.h>
#include <IWatchdog.h>
#include <app.h>
#include <vector>
#include <simple_dronecanmessages.h>

// set up your parameters here with default values. NODEID should be kept
std::vector<DroneCAN::parameter> custom_parameters = {
    { "NODEID", UAVCAN_PROTOCOL_PARAM_VALUE_INTEGER_VALUE, 69,  0, 127 },
    { "PARM_1", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_2", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_3", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_4", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_5", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_6", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
    { "PARM_7", UAVCAN_PROTOCOL_PARAM_VALUE_REAL_VALUE,   0.0f, 0.0f, 100.0f },
};

DroneCAN dronecan;

uint32_t looptime = 0;

/*
This function is called when we receive a CAN message, and it's accepted by the shouldAcceptTransfer function.
We need to do boiler plate code in here to handle parameter updates and so on, but you can also write code to interact with sent messages here.
*/
static void onTransferReceived(CanardInstance *ins, CanardRxTransfer *transfer)
{

    // switch on data type ID to pass to the right handler function
    // if (transfer->transfer_type == CanardTransferTypeRequest)
    // check if we want to handle a specific service request
    switch (transfer->data_type_id)
    {

    case UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_ID:
    {
        uavcan_equipment_ahrs_MagneticFieldStrength pkt{};
        uavcan_equipment_ahrs_MagneticFieldStrength_decode(transfer, &pkt);
        break;
    }
    }

    DroneCANonTransferReceived(dronecan, ins, transfer);
}

/*
For this function, we need to make sure any messages we want to receive follow the following format with
UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_ID as an example
 */
static bool shouldAcceptTransfer(const CanardInstance *ins,
                                 uint64_t *out_data_type_signature,
                                 uint16_t data_type_id,
                                 CanardTransferType transfer_type,
                                 uint8_t source_node_id)

{
    if (transfer_type == CanardTransferTypeBroadcast)
    {
        // Check if we want to handle a specific broadcast packet
        switch (data_type_id)
        {
        case UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_ID:
        {
            *out_data_type_signature = UAVCAN_EQUIPMENT_AHRS_MAGNETICFIELDSTRENGTH_SIGNATURE;
            return true;
        }
        }
    }

    return false || DroneCANshoudlAcceptTransfer(ins, out_data_type_signature, data_type_id, transfer_type, source_node_id);
}

void setup()
{
    // to use debugging tools, remove app_setup and set FLASH start from 0x800A000 to 0x8000000 in ldscript.ld
    // this will over-write the bootloader. To use the bootloader again, reflash it and change above back.
    app_setup(); // needed for coming from a bootloader, needs to be first in setup
    Serial.begin(115200);
    
    dronecan.version_major = 1;
    dronecan.version_minor = 0;
    dronecan.init(
        onTransferReceived, 
        shouldAcceptTransfer, 
        custom_parameters,
        "Beyond Robotix Node"
    );

    IWatchdog.begin(2000000); // if the loop takes longer than 2 seconds, reset the system

    // an example of getting and setting parameters within the code
    dronecan.setParameter("PARM_1", 69);
    Serial.print("PARM_1 value: ");
    Serial.println(dronecan.getParameter("PARM_1"));

    while (true)
    {
        const uint32_t now = millis();

        // send our battery message at 10Hz
        // Don't use delay() since we need to call dronecan.cycle() as much as possible
        if (now - looptime > 100)
        {
            looptime = millis();

            // collect MCU core temperature data
            int32_t vref = __LL_ADC_CALC_VREFANALOG_VOLTAGE(analogRead(AVREF), LL_ADC_RESOLUTION_12B);
            int32_t cpu_temp = __LL_ADC_CALC_TEMPERATURE(vref, analogRead(ATEMP), LL_ADC_RESOLUTION_12B);

            // construct dronecan packet
            uavcan_equipment_power_BatteryInfo pkt{};
            pkt.voltage = analogRead(PA1);
            pkt.current = analogRead(PA0);
            pkt.temperature = cpu_temp;

            sendUavcanMsg(dronecan.canard, pkt);
        }

        dronecan.cycle();
        IWatchdog.reload();
    }
}

void loop()
{
    // Doesn't work coming from bootloader ? use while loop in setup
}
```

