---
title: Module Software Library
sidebar_position: 2
---


# Module Support Software Library 

The Software library to support modules is available at http://github.com/DraigDeg/module-library

This guide describes how to interact with it. 

We'll start with the basic usage, the setup, then using it with a specific use case. 

## Basic Usage


Get the library software included in your software. 

### PlatformIO setup.

We use PlatformIO to handle the build of our modules so that's what we'll describe here. 

in `platformio.ini` add the following

```properties
lib_deps = https://github.com/DraigDeg/draigdeg-module-library.git#v1.0.0
```

### Arduino IDE setup.

You can use Arduino IDE - Get the library as a zip and add it to your library folder. 

:::warning

Untested. This needs verifying and improving.

:::


## Setup the library.

**1. Include the header**

At the very top of the file include the C header

```CPP
#include <DraigDegModule.h>
```

**2. Create an instance of a `DraigDegModule` object**


Somewhere near the top of the file, before your functions, add...

```CPP
DraigDegModule module;
```

**3. Setup the serial ports**

in Arduino's `setup()` function you'll need things that look similar to the following.

```CPP
Serial.begin(9600);  // this is the USB port (programming/console) on our Xiao-based module
Serial1.begin(9600); // this is the hardware UART on two pins coupled to the controller port

module.begin(Serial1); // give the UART to the module library.
module.debug(Serial);  // Optionally add some debugging here.

```

**4. Setup the module metadata**

```CPP
const byte MANUFACTURER_ID = 999; // set this to your own organisation or manufacturer ID - ignore if private data.  
const byte MODULE_MODEL_ID = 1;   // set this to your own equipment model code - can be anything if private data.
const byte MODULE_HW_REV = 1;     // update these as you go
const byte MODULE_SW_REV = 1;     // update these as you go
const byte FEATURES = 0;          // reserved
module.setModuleInfo(MANUFACTURER_ID, MODULE_MODEL_ID, MODULE_HW_REV, MODULE_SW_REV, FEATURES);
```

**5. Describe your sensors!**
Tell the module what kind of sensor channels, instruments and encodings you will be supplying

Still in the `setup()` function

```CPP

Channel* ch = module.addChannel(1);
if (ch) {
    ch->addMeasurement(MEASUREMENT_VOLTAGE, ENCODING_UINT16, 0.0);
    ch->enable();
}
```

In the above snippet, we create a channel (you can have many), this is channel 1.
Channel 1 is then allocated one measurement: a voltage reading.
We also specifiy the encoding we want the controller to use to get these values over the air. 

:::info
The data that flows over the serial port should be human readable - so we use ASCII.  But numbers in ASCII takes up a lot of space when you try and transmit them on a low-bandwidth carrier such as LoRaWAN. Because the controller doesn't know (or really care) what you want to send, we must supply a hint for the encoding. 
By picking the encoding, we can determine the resolution that makes sense for the data.  unsigned and signed integers are supported, as are floats but they are quite byte-hungry. We also have a few custom ones for "normal" temperatures at 2 decimal places (which we use here as encoding number 42) and relative humidity at encoding 41 which gives 0.5% resolution. 

You can create your own. #opensource! 
:::

**6. Create a function to read your data** 

Supply the measurements with the right values.

Above the `setup()` function create a function of your own...

It must be void and with no arguments. 

```CPP
void readAll() {
  module.invalidate(); // clear the values
  Channel* ch = module.getChannel(1);
  if (ch) {
      // Read SHT
      sensors_event_t rh, temp;
      bool ok = sht4.getEvent(&rh, &temp);// populate temp and humidity objects with fresh data
      Serial.print("Reading OK ");
      Serial.println(ok);
      if(ok){
        Serial.println(temp.temperature);
        Serial.println(rh.relative_humidity);
        ch->updateMeasurement(0, temp.temperature);
        ch->updateMeasurement(1, rh.relative_humidity);
      }
    }
}
```

In the code snippet above we can see a routine which reads from an SHT40 sensor - but it could be whatever you want. 
The previously defined measurements on channel 1 get updated.

**6. Register this as a callback**

Back in `setup()` at the end, add the following

```CPP
module.registerCollectMeasurements(readAll);
```

**7. Service the UART**

All that's required now is to service the UART in the main loop

```CPP
void loop() {
    // Process incoming commands
    if(!module.processSerial()){
        delay(10);
    }
}
```

And the library will take care of itself!