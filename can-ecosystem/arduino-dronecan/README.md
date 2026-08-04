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
This documentation is written for the 2.X version of the library. If you're on a different version, the main.cpp file of your version should demonstrate the interface of that version. The 1.X interface is different in a few places - see [Upgrading from 1.X](./#upgrading-from-1.x) at the bottom of this page.
{% endhint %}

Github Repository:

{% embed url="https://github.com/BeyondRobotix/Arduino-DroneCAN" %}

## Supported hardware

The same application code builds for all of our CAN nodes. Which board you pick in PlatformIO decides the pinmap, the linker script and which CAN driver gets compiled in.

<table><thead><tr><th width="180">Board</th><th width="120">MCU</th><th width="100">Flash</th><th width="100">CAN FD</th><th>PlatformIO environment</th></tr></thead><tbody><tr><td>Micro Node</td><td>STM32L431</td><td>256 KB</td><td>No</td><td><code>Micro-Node-App</code></td></tr><tr><td>MicroNode+</td><td>STM32H723</td><td>1 MB</td><td>Yes</td><td><code>Micro-Node-Plus-App</code></td></tr><tr><td>Core Node</td><td>STM32H743</td><td>2 MB</td><td>Yes</td><td><code>Core-Node-App</code></td></tr></tbody></table>

## Installation

There are a few ways of working with Arduino Code, we recommend the following steps for seamless integration with the project.

1. Install Visual Studio Code [https://code.visualstudio.com/download](https://code.visualstudio.com/download)
2. Install the PlatformIO extension [https://platformio.org/install/ide?install=vscode](https://platformio.org/install/ide?install=vscode)
3. Download the project [https://github.com/BeyondRobotix/Arduino-DroneCAN/releases](https://github.com/BeyondRobotix/Arduino-DroneCAN/releases)
4. Select the environment for your board from the PlatformIO status bar
5. Connect your STLINK to your CAN node
6. Press upload!



There are no submodules any more, so a plain clone is all you need:

```
git clone https://github.com/BeyondRobotix/Arduino-DroneCAN.git
```

The library and the board definitions live in their own repositories and are fetched by PlatformIO on the first build, pinned to a version in `platformio.ini`:

```ini
[env]
platform = https://github.com/BeyondRobotix/br_platformio_hwdef.git#v1.1
lib_deps = https://github.com/BeyondRobotix/libArduinoDroneCAN.git#v2.0.0
```

The first build takes a couple of minutes while these download. To move to a newer library, change the `#vX.Y.Z` tag and delete `.pio/libdeps/` so PlatformIO re-fetches it. To pick up a new platform release, run `pio pkg uninstall --platform br-stm32 -g`.

## Bootloader details

ArduinoDroneCAN uses our own bootloader, [BR\_bootloader](https://github.com/BeyondRobotix/BR_bootloader), to allow app updates over CAN. The standard AP\_Bootloader cannot be used.

You don't need to flash it separately. The bootloader binary for each board ships inside the `br_platformio_hwdef` platform, and any environment ending in `-App` flashes the bootloader and your app in one go on upload. A brand new board is fully provisioned by a single `pio run -t upload`.



## Breakpoint debugging

This works as standard in PlatformIO debugging! It previously needed a special flash without the bootloader, but this is no longer an issue.


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

If your node only needs to **send** messages, use the short form of init(). You just pass a parameter list and the name of the node for NodeInfo, and the library wires up the receive callbacks for you:

```cpp
dronecan.version_major = 1;
dronecan.version_minor = 0;
dronecan.init(
    custom_parameters,
    "Beyond Robotix Node"
);
```

If your node needs to **receive** messages, pass your own two callback functions as the first two arguments (we explain these later):

```cpp
dronecan.init(
    onTransferReceived, 
    shouldAcceptTransfer, 
    custom_parameters,
    "Beyond Robotix Node"
);
```

Both forms take the same optional arguments after the node name, all of which have sensible defaults:

<table><thead><tr><th width="180">Argument</th><th width="220">Default</th><th>What it does</th></tr></thead><tbody><tr><td><code>mode</code></td><td><code>CanMode::Classic</code></td><td>Classic CAN 2.0B, or CAN FD on H7 boards. See <a href="#can-fd">CAN FD</a></td></tr><tr><td><code>port</code></td><td><code>CanPort::PORT1</code></td><td>Which physical CAN port this instance uses. See <a href="#two-can-ports">Two CAN ports</a></td></tr><tr><td><code>storage_page</code></td><td><code>-1</code> (board default)</td><td>First flash page of the parameter storage region. Leave it alone unless you know you need to move it</td></tr><tr><td><code>persist_on_set</code></td><td><code>true</code></td><td>Whether a parameter written over CAN saves itself. See <a href="#parameters">parameters</a></td></tr></tbody></table>

#### Watchdog

```cpp
IWatchdog.begin(2000000);
```

Our program requires starting the watchdog, remove it at your own peril! (it's started in the bootloader and from our testing, starting it here again is best)&#x20;



### DroneCAN looping

The normal Arduino `loop()` function works. Older versions of this library needed all of your looping code inside a `while(true)` at the end of `setup()` - that restriction is gone. Sketches written the old way still run fine, so you don't have to rewrite them.

```cpp
void loop()
{
    // your code here

    dronecan.cycle();
    IWatchdog.reload();
}
```

#### Fixed interval loops

The following code sets us up to run our if statement at 10Hz, checking if 100ms has passed since our last call. This is because we want to only send our CAN message at 10Hz and we want to call our "dronecan.cycle()" function to be called as much as possible to ensure CAN messages are send and received in a timely manner. We want to avoid delay() as much as possible!

```cpp
const uint32_t now = millis();

    // send our battery message at 10Hz
    if (now - looptime > 100)
    {
        looptime = now;
```



#### Sensor reading

Next, we want to read in our sensor value. This could be from anything, a current monitor, position sensor.. in this example, we read in the temperature of our STM32 processor. You would have had to have initialised your sensor in setup() of course.

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
sendUavcanMsg(dronecan, pkt);

// or, to set the transfer priority yourself
sendUavcanMsg(dronecan, pkt, CANARD_TRANSFER_PRIORITY_LOW);
```

{% hint style="info" %}
Pass the **dronecan object**, not `dronecan.canard`. The older `sendUavcanMsg(dronecan.canard, pkt)` form still compiles, but it doesn't know about the node's CAN FD setting, so on an FD node it will send that message as a classic CAN frame. See [CAN FD](#can-fd).
{% endhint %}



For messages which are not in this type list:

```python
messages = [
    "dronecan.sensors.hygrometer.Hygrometer",
    "dronecan.sensors.magnetometer.MagneticFieldStrengthHiRes",
    "dronecan.sensors.rc.RCInput",
    "dronecan.sensors.rpm.RPM",
    "uavcan.equipment.actuator.ArrayCommand",
    "uavcan.equipment.actuator.Status",
    "uavcan.equipment.ahrs.MagneticFieldStrength",
    "uavcan.equipment.ahrs.MagneticFieldStrength2",
    "uavcan.equipment.ahrs.RawIMU",
    "uavcan.equipment.ahrs.Solution",
    "uavcan.equipment.air_data.AngleOfAttack",
    "uavcan.equipment.air_data.IndicatedAirspeed",
    "uavcan.equipment.air_data.RawAirData",
    "uavcan.equipment.air_data.Sideslip",
    "uavcan.equipment.air_data.StaticPressure",
    "uavcan.equipment.air_data.StaticTemperature",
    "uavcan.equipment.air_data.TrueAirspeed",
    "uavcan.equipment.camera_gimbal.AngularCommand",
    "uavcan.equipment.camera_gimbal.GEOPOICommand",
    "uavcan.equipment.camera_gimbal.Status",
    "uavcan.equipment.device.Temperature",
    "uavcan.equipment.esc.RawCommand",
    "uavcan.equipment.esc.RPMCommand",
    "uavcan.equipment.esc.Status",
    "uavcan.equipment.esc.StatusExtended",
    "uavcan.equipment.gnss.Auxiliary",
    "uavcan.equipment.gnss.Fix",
    "uavcan.equipment.gnss.Fix2",
    "uavcan.equipment.gnss.RTCMStream",
    "uavcan.equipment.hardpoint.Command",
    "uavcan.equipment.hardpoint.Status",
    "uavcan.equipment.ice.FuelTankStatus",
    "uavcan.equipment.ice.reciprocating.Status",
    "uavcan.equipment.indication.BeepCommand",
    "uavcan.equipment.indication.LightsCommand",
    "uavcan.equipment.power.BatteryInfo",
    "uavcan.equipment.power.CircuitStatus",
    "uavcan.equipment.power.PrimaryPowerSupplyStatus",
    "uavcan.equipment.range_sensor.Measurement",
    "uavcan.equipment.safety.ArmingStatus",
]
```



we have to do the full boilerplate for a UAVCAN/DroneCAN message:

```cpp
uint8_t buffer[UAVCAN_EQUIPMENT_POWER_BATTERYINFO_MAX_SIZE];
uint32_t len = uavcan_equipment_power_BatteryInfo_encode(&pkt, buffer
    CANARD_TAO_ENCODE_ARG(true)
);
static uint8_t transfer_id;
CanardTxTransfer transfer_object = {
    .transfer_type = CanardTransferTypeBroadcast,
    .data_type_signature = UAVCAN_EQUIPMENT_POWER_BATTERYINFO_SIGNATURE,
    .data_type_id = UAVCAN_EQUIPMENT_POWER_BATTERYINFO_ID,
    .inout_transfer_id = &transfer_id,
    .priority = CANARD_TRANSFER_PRIORITY_LOW,
    .payload = buffer,
    .payload_len = (uint16_t)len,
};
canardBroadcastObj(&dronecan.canard, &transfer_object);
```

`transfer_id` has to be `static` (or global) - libcanard advances it for you after each successful send, so it needs to survive between calls.

{% hint style="info" %}
Older versions of this documentation showed the `canardBroadcast()` call with a long argument list. That function still exists, but its signature grows extra arguments on CAN FD builds, so it won't compile for the H7 boards. The `canardBroadcastObj()` form above builds everywhere. `CANARD_TAO_ENCODE_ARG()` is a helper macro from the library that expands to nothing on builds where the encoder doesn't take a tail-array-optimisation argument.
{% endhint %}



#### End of the loop

```cpp
dronecan.cycle();
IWatchdog.reload();
```

Outside our fixed interval if statment, we call our dronecan.cycle() method, which is required, and we also reset our watchdog timer. &#x20;



### Reading in CAN packets

onTransferReceived and shouldAcceptTransfer are defined in our program and passed to dronecan.init. These functions are used to recieve any DroneCAN messages. If your node only sends messages you can leave them out entirely and use the short form of init() - the library then handles the standard DroneCAN traffic on its own. We've put an example of decoding a Magnetometer message below. in onTransferRecieved, you can manipulate the recieved packet and use gobal variables or parameters etc to transfer information out of this function. DroneCANonTransferReceived must always be called at the end of this function for our library to work.

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

    return false || DroneCANshouldAcceptTransfer(ins, out_data_type_signature, data_type_id, transfer_type, source_node_id);
}
```



### Other things

#### parameters

```cpp
std::vector<DroneCAN::parameter> custom_parameters = {
    { "NODEID", DroneCAN::INT,  100,  1,    127 },
    { "PARM_1", DroneCAN::REAL, 0.0f, 0.0f, 100.0f },
    { "PARM_2", DroneCAN::REAL, 0.0f, 0.0f, 100.0f },
    { "PARM_3", DroneCAN::REAL, 0.0f, 0.0f, 100.0f },
};
```

We passed the custom\_parameters object into dronecan.init. This parameters object needs "NODEID" in the list since we use that. The first value after the type dictates the default value, second the minimum for the parameter and the last the maximum. These min/maxes may or may not be shown in Mission planner etc and may or may not make any functional difference.

`DroneCAN::INT`, `DroneCAN::REAL` and `DroneCAN::BOOL` are short aliases for the `UAVCAN_PROTOCOL_PARAM_VALUE_*` constants. The long names still work if you prefer them.



```cpp
dronecan.setParameter("PARM_1", 69);
Serial.print("PARM_1 value: ");
Serial.println(dronecan.getParameter("PARM_1"));
```

You can set and retrieve parameters easily. parameters get saved to flash so they'll be recalled on next boot.&#x20;

#### CAN FD

{% hint style="info" %}
CANFD support is experimental. So far bench tests have shown 2mpbs functioning, and some success with 4mpbs.
{% endhint %}


The H7 boards (MicroNode+ and Core Node) support CAN FD. Pass a `CanMode` to init() to turn it on:

```cpp
dronecan.init(custom_parameters, "Beyond Robotix Node", DroneCAN::CanMode::FD4X);
```

The arbitration bitrate is always 1 Mbps. The mode picks the data-phase multiplier on top of that, so match it to the autopilot's `CAN_Px_FDBITRATE` parameter:

<table><thead><tr><th width="220">CanMode</th><th width="180">Data phase bitrate</th><th>ArduPilot CAN_Px_FDBITRATE</th></tr></thead><tbody><tr><td><code>CanMode::Classic</code> (default)</td><td>n/a - CAN 2.0B</td><td>-</td></tr><tr><td><code>CanMode::FD2X</code></td><td>2 Mbps</td><td>2</td></tr><tr><td><code>CanMode::FD4X</code> (also <code>CanMode::FD</code>)</td><td>4 Mbps</td><td>4</td></tr><tr><td><code>CanMode::FD8X</code></td><td>8 Mbps</td><td>8</td></tr></tbody></table>

Once a node is initialised in an FD mode, `sendUavcanMsg(dronecan, pkt)` sends FD frames automatically. This is why you should pass the dronecan object rather than `dronecan.canard` - the `canard` form has no way to know the node is in FD mode and will send a classic frame instead.

The Micro Node's L431 has no FD hardware. Asking for an FD mode in a Micro Node build is a compile error rather than a runtime surprise.

#### Two CAN ports

The H7 boards have two CAN peripherals, and you can run both from one firmware by creating two `DroneCAN` objects. Each is an independent node with its own node ID, its own parameter values and its own transfer IDs:

```cpp
DroneCAN can1;
DroneCAN can2;

void setup()
{
    app_setup();
    IWatchdog.begin(2000000);

    can1.init(params_port1, "Beyond Robotix Node/Port1",
              DroneCAN::CanMode::FD, DroneCAN::CanPort::PORT1);
    can2.init(params_port2, "Beyond Robotix Node/Port2",
              DroneCAN::CanMode::FD, DroneCAN::CanPort::PORT2);
}

void loop()
{
    can1.cycle();
    can2.cycle();
    IWatchdog.reload();
}
```

Give the two instances different default NODEIDs so they don't collide on a shared bus. Both instances share the same pair of flash pages for parameter storage - each record is tagged with which instance it belongs to - so the second port costs no extra flash.

There is also `CanPort::BOTH`, which bridges the two ports into a single node: messages go out on both ports and received messages are merged. This is the redundant-interface setup ArduPilot expects when you wire one node to two buses.

The full working version of this is the `Dual_CAN` example. The Micro Node has a single CAN port and ignores the port argument.



### Full example!

This is `src/main.cpp` from the repository. It's a send-only node, so it uses the short form of init(). If you need to receive messages, add the two callback functions from [Reading in CAN packets](#reading-in-can-packets) and pass them as the first two arguments to init().

```cpp
#include <Arduino.h>
#include <dronecan.h>

// set up your parameters here with default values. NODEID should be kept
std::vector<DroneCAN::parameter> params_port1 = {
    {"NODEID", DroneCAN::INT,  100,  1,    127},
    {"PARM_1", DroneCAN::REAL, 0.0f, 0.0f, 100.0f},
    {"PARM_2", DroneCAN::REAL, 0.0f, 0.0f, 100.0f},
};

DroneCAN can1;

uint32_t looptime1 = 0;

void setup()
{
    app_setup();              // needed for coming from a bootloader, needs to be first in setup
    IWatchdog.begin(2000000); // if the loop takes longer than 2 seconds, reset the system
    Serial.begin(115200);

    can1.init(params_port1, "Beyond Robotix Node",
              DroneCAN::CanMode::Classic, DroneCAN::CanPort::PORT1);

    Serial.print("Port1 NODEID: "); Serial.println(can1.getParameter("NODEID"));
}

void loop()
{
    const uint32_t now = millis();

    // send our battery message at 10Hz
    // Don't use delay() since we need to call can1.cycle() as much as possible
    if (now - looptime1 > 100)
    {
        looptime1 = now;

        // collect MCU core temperature data
        int32_t vref     = __LL_ADC_CALC_VREFANALOG_VOLTAGE(analogRead(AVREF), LL_ADC_RESOLUTION_12B);
        int32_t cpu_temp = __LL_ADC_CALC_TEMPERATURE(vref, analogRead(ATEMP), LL_ADC_RESOLUTION_12B);

        // construct dronecan packet
        uavcan_equipment_power_BatteryInfo pkt{};
        pkt.voltage     = analogRead(PA1);
        pkt.current     = analogRead(PA0);
        pkt.temperature = cpu_temp;

        sendUavcanMsg(can1, pkt, CANARD_TRANSFER_PRIORITY_LOW);
    }

    can1.cycle();
    IWatchdog.reload();
}
```

`<dronecan.h>` pulls in everything the library needs, so the separate `<IWatchdog.h>`, `<app.h>`, `<vector>` and `<simple_dronecanmessages.h>` includes older examples carried are no longer required.

## Upgrading from 1.X

If you're moving an existing sketch from a 1.X release, these are the things that changed:

<table><thead><tr><th width="260">1.X</th><th>2.X</th></tr></thead><tbody><tr><td>Clone with <code>--recurse-submodules</code></td><td>No submodules. PlatformIO fetches the library and board definitions from <code>platformio.ini</code></td></tr><tr><td><code>Micro-Node-Bootloader</code> environment</td><td><code>Micro-Node-App</code>, plus <code>Core-Node-App</code> and <code>Micro-Node-Plus-App</code> for the H7 boards</td></tr><tr><td>Loop code inside <code>while(true)</code> in <code>setup()</code></td><td>Normal Arduino <code>loop()</code>. The old form still works</td></tr><tr><td>init() always needed the two callback functions</td><td>Callbacks are optional - omit them if your node only sends</td></tr><tr><td><code>sendUavcanMsg(dronecan.canard, pkt)</code></td><td><code>sendUavcanMsg(dronecan, pkt)</code>, so CAN FD is handled for you</td></tr><tr><td><code>DroneCANshoudlAcceptTransfer</code></td><td><code>DroneCANshouldAcceptTransfer</code> (spelling fixed)</td></tr><tr><td><code>UAVCAN_PROTOCOL_PARAM_VALUE_INTEGER_VALUE</code></td><td><code>DroneCAN::INT</code> and friends. The long names still work</td></tr><tr><td>Parameters needed "Store all" to persist</td><td>Parameters written over CAN save themselves</td></tr></tbody></table>

Reflashing a 1.X node with 2.X firmware resets its parameters to the defaults in your code, once. Note down anything you've changed from default before you upgrade.

