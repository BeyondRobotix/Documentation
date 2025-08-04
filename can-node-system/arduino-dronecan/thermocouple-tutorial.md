---
description: >-
  A demo of the Arduino DroneCAN library with Beyond Robotix CAN node, using the
  MCP9600 thermocouple reader.
icon: building-columns
---

# Thermocouple Tutorial

## Intro

Do you want to integrate a sensor into Ardupilot or PX4? Arduino DroneCAN and the Beyond Robotix CAN node let you do that very quickly. This tutorial runs through integrating the Adafruit MCP9600 thermocouple sensor, resulting in us being able to send temperature messages over DroneCAN. The big advantage of using the Arduino framework is access to Arduino libraries. Almost always, there are libraries available for the sensor you want to use. This saves development time. The Arduino framework is also simple to work with, and the Platformio + VS code platform allows easy development from small to complicated projects.

<figure><img src="../../.gitbook/assets/title photo-Photoroom (1).jpg" alt="" width="375"><figcaption></figcaption></figure>

Using custom firmware allows lots of options, such as integrating a battery monitor + thermocouple and sending all the information with one device in one message. You could action tasks based on the thermocouple onboard the node, e.g. opening a hatch if the battery is getting too hot.

You could integrate GPS receivers, new airspeed sensors, fuel sensors, gas sensors, ESCs and servo nodes.

## Hardware Requirements

* Beyond Robotix CAN Node Dev Kit - [https://www.beyondrobotix.com/products/micro-can-node-development-bundle](https://www.beyondrobotix.com/products/micro-can-node-development-bundle)
* Flight controller with a CAN bus (Or a CAN 'Sniffer' like the [Zubax Babel](https://shop.zubax.com/products/zubax-babel))
* Windows PC  (Other OS work with minor adjustments)
* [Adafruit MCP9600](https://www.adafruit.com/product/4101) + [Thermocouple](https://www.adafruit.com/product/270)



## Getting started

To start, we should download the latest release of Arduino DroneCAN from:

{% embed url="https://github.com/BeyondRobotix/Arduino-DroneCAN/releases" %}

The code we will run through is in examples/thermocouple-mcp9600 which is here:

{% embed url="https://github.com/BeyondRobotix/Arduino-DroneCAN/tree/master/examples/thermocouple-MCP9600" %}

We’ll need to download a few things to work with the software. We’ve got these setup instructions on our documentation site here:

{% embed url="https://beyond-robotix.gitbook.io/docs/can-node-system/arduino-dronecan" %}

We’ll also need to get our hardware setup. You’ll need a DroneCAN compatible flight controller or sniffer. We’ll use a Cube Orange. Then connect your Beyond Robotix CAN node to the flight controller via a CAN cable. Also, connect your STLINK to the debug port on the CAN node. Lastly, ensure the switch next to the debug port (`SW1`) is set to '1'.

<figure><img src="../../.gitbook/assets/hardware-Photoroom.jpg" alt="" width="375"><figcaption></figcaption></figure>

With the repository downloaded and our tools installed, we can build the default example to make sure it’s all working correctly. The default example sends a BatteryInfo message with values from ADC pins PA0 and PA1 as well as MCU temperature.

<figure><img src="../../.gitbook/assets/Picture1.png" alt="" width="563"><figcaption></figcaption></figure>

You should then see “SUCCESS” shown in the terminal. If you don’t, make sure your STLINK is connected correctly and power is being given to the CAN node via the flight controller.&#x20;

<figure><img src="../../.gitbook/assets/Picture3 (1).png" alt="" width="563"><figcaption></figcaption></figure>

We can now see what CAN messages are being sent by the Node. We’ll use DroneCAN GUI tool in this example, however, Mission Planner can also be used for CAN packet inspection.&#x20;

{% embed url="https://dronecan.github.io/GUI_Tool/Overview/" %}

<figure><img src="../../.gitbook/assets/Picture4.png" alt="" width="310"><figcaption></figcaption></figure>

Once connected, we’ll see our Node showing in the list, (you may need to set the Local node ID, press the tick in the top left)

<figure><img src="../../.gitbook/assets/Picture5.png" alt="" width="563"><figcaption></figcaption></figure>

We can see the example parameters in Node properties:

<figure><img src="../../.gitbook/assets/Picture7.png" alt="" width="375"><figcaption></figcaption></figure>

And we can also see the battery message being sent (which can be found in Tools > Subscriber)

<figure><img src="../../.gitbook/assets/Picture8.png" alt="" width="563"><figcaption></figcaption></figure>

We can see the temperature field which responds to the built in MCU temperature sensor. “Voltage” and “current” are showing the raw ADC values from PA1 and PA0 respectively.

## Getting MCP9600 data

We’ve chosen to use the Adafruit library, although there are a few MCP9600 libraries to choose from. Download the latest release, unzip and add it to the “lib” folder in the Arduino DroneCAN repository. Make sure there are no nested folders! The library can also be installed from the platformio library manager. The “BusIO” library is also needed. Platformio may install this for you.

{% embed url="https://github.com/adafruit/Adafruit_MCP9600/releases/tag/2.0.4" %}

{% embed url="https://github.com/adafruit/Adafruit_BusIO/releases/tag/1.17.2" %}

Let's have a look at their example to understand how to use their library.

{% embed url="https://github.com/adafruit/Adafruit_MCP9600/blob/master/examples/mcp9600_test/mcp9600_test.ino" %}

They import the files, set the I2C address and setup the `mcp` object. We had to set the address to 0x66 for the MCP9600 we had.

```cpp
#include <Wire.h>
#include <Adafruit_I2CDevice.h>
#include <Adafruit_I2CRegister.h>
#include "Adafruit_MCP9600.h"

#define I2C_ADDRESS (0x67)

Adafruit_MCP9600 mcp;
```

We then call `.begin` method

```cpp
 /* Initialise the driver with I2C_ADDRESS and the default I2C bus. */
if (! mcp.begin(I2C_ADDRESS)) {
  Serial.println("Sensor not found. Check wiring!");
  while (1);
}
```

Then, they have a bunch of other options but the two we're interested in for the basics:

```cpp
 // set our thermocouple type
 mcp.setThermocoupleType(MCP9600_TYPE_K);
 
 // take a temperature measurement
 mcp.readThermocouple()
```

So, with this information, we can now write our DroneCAN application.

## Writing our app

Now, we go to `main.cpp` in Arduino DroneCAN and add the code from above to our  `setup()` function. We have provided an example for you to follow:

{% embed url="https://github.com/BeyondRobotix/Arduino-DroneCAN/blob/master/examples/thermocouple-MCP9600/main.cpp" %}

Quick aside, in many arduino projects, you might use `delay()`. This can cause problems with Arduino DroneCAN as ot needs to perform functions in the background. So, instead of using `delay()`, setup a statment that only runs once a timer counts to a given time, in this example 1000ms. When not meeting this condition, it always calls "dronecan.cycle()" which processes CAN frames in the background and "IWatchdog.reload()" which ensures our watchdog doesn't reset.

In this example, when we are unable to connect to the MCP9600, we want to keep resending the message every second over CAN and serial to say we have a problem. Instead of using a delay, we use an `if` statement which waits until a time has elapsed while still allowing the background tasks to run.

<mark style="color:$warning;">Bad Code - Don't Use</mark> <mark style="color:$warning;"></mark><mark style="color:$warning;">`delay()`</mark>

<pre class="language-cpp"><code class="lang-cpp"><strong>if (!mcp.begin(I2C_ADDRESS))
</strong>{
    uint32_t deadloop = 0;
    while (1)
    {
        dronecan.debug("MCP9600 not found", 0);
        Serial.println("Sensor not found. Check wiring!");

        dronecan.cycle();
        IWatchdog.reload();
        
        delay(1000);
    }
}

// Set the thermocouple type
mcp.setThermocoupleType(MCP9600_TYPE_K);
</code></pre>

<mark style="color:$success;">Good Code - use</mark> <mark style="color:$success;"></mark><mark style="color:$success;">`if`</mark> <mark style="color:$success;"></mark><mark style="color:$success;">as timer</mark>

```cpp
if (!mcp.begin(I2C_ADDRESS))
{
    uint32_t deadloop = 0;
    while (1)
    {
        const uint32_t now = millis();
        if (now - deadloop > 1000)
        {
            deadloop = millis();
            dronecan.debug("MCP9600 not found", 0);
            Serial.println("Sensor not found. Check wiring!");
        }
        dronecan.cycle();
        IWatchdog.reload();
    }
}

// Set the thermocouple type
mcp.setThermocoupleType(MCP9600_TYPE_K);

```

Another point is we don't use the `void loop()` that arduino uses normally. Instead we have a `while(true){` running at the end of the `void setup()` function.

```cpp
void setup()
{
    # Set up Code
    
    while(true){
        # Looping Code
        
    }  
}  
```

Now, in the example, we've written in some logic which lets you send either a temperature packet or a battery packet depending on some parameters. At the core of it, we send a DroneCAN message like this:

```cpp
uavcan_equipment_device_Temperature pkt{};

pkt.temperature = mcp.readThermocouple();
pkt.device_id = device_id;

sendUavcanMsg(dronecan.canard, pkt);
```

We can see that we read in our thermocouple measurement, and put it into a Temperature packet. You can see the packets availible here:

{% embed url="https://dronecan.github.io/Specification/7._List_of_standard_data_types/" %}

Although there are lots of different types of packets, not all are actually understood by PX4 or Ardupilot, so you'll have to do some research on the appropriate packet to use. If you get stuck you can contact us at **admin@beyondrobotix.com**

We also set the device\_id, which we pull from a parameter in a different part of the loop. This is used by Ardupilot to identify which battery instance to put the temperature into.

We then do our "sendUavcanMsg" which queues our packet.&#x20;

And there we go! If you compile our example, you'll see either battery or temperature packets coming through depending on the parameters set.

We can create fairly complex features easily : - ) If you use the library and/or CAN node in your project, we'd be keen to hear about it! contact us at **admin@beyondrobotix.com**
