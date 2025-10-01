---
sidebar_label: 'Data Encoding'
sidebar_position: 3
---

# Data Encoding

Data is encoded in different ways at different points. 

Controller-to-Module is encoded on the wire in ASCII for readability. 

Controller-to-Cloud is encoded in various binary encoding formats to ensure the wireless communications are efficient. 

The binary encoding are self-described by the modules for each channel measurement. 

## Channels

A module may provide multiple modules of similar or different types of measurements. 

Example 1: a multi-depth temperature sensor with multiple channels, each providing a single temperature reading.

Example 2: A multi-room environment sensor, with multiple instruments, each channel providing a temperature and a humidity reading.

Example 3: An industrial measurement sensor, with each channel providing a different measurement from unrelated instruments. 

Channels are numeric, with instrumentation channels starting from 1. Channel 0 is reserved to refer to the module itself. 

## Measurements

A Measurement is a specific thing being measured.
Each measurement is general but mapped by the channel to a thing, or a bit of a thing, in the real world.

Measurements included properties such as temperature, humidity, pressure, but also readings such as voltage, current, or digital states (on/off)

These are always numeric, and are encoded in a manner that is most efficient for transmission.

A module can choose different encodings for each specific measurement on each channel, depending on the value range. 

## Descriptors

To reduce configuration, a module is self describing. When interrogated it must respond with details that decribe the data it will be emitting.

When data is emitted, it also describes the measurement and encoding. 

A module description defines the number of channels it emits data for.

A channel description defines the measurements about which it emits data for, and their numeric encodings.


## Measurement Type

The following list of measurement types may be expanded as required.
A measurement type has a single unsigned byte identifier, and a measuremnt unit (and an optional standard resolution if not whole numbers).

For example - when measuring voltage it is typical to measure and transmit in millivolts. The encoding specified in each measurement will provide the precision and resolution. 

| ID | Name          | unit (and option resolution) |
| 1  | Temperature   | Celcius     |
| 2  | Humidity      | % Relative  |
| 3  | Voltage       | Millivolts  |
| 4  | Current       | Milliamps   |
| 5  | Digital       | 1 or 0      |

This table can be expanded as required. 


## Encoding Type

Each application of measurement will require a specific or tolerable level of precision. 
Some application may only need whole numbers, but other may need many decimal places of precision. 
In addition to specifiying the measurement type, a measurement is always described and transmitted with its encoding. 
At capture time a module may determine the most efficient encoding to use. 

A encoding type has a single unsigned byte identifier, a length in bytes, and a name and a descrption of how the data is encoded. 
Many are standard types of binary coding, E.g. integers of different numbr of bytes, twos-complement signed integers of various numbers of bytes, IEEE754 floats of different numbers of bytes.
We also define some specific encodings which aid efficiency for certain value ranges that may be applicable. 

| ID |  Type | numerical type | Number of Bytes | Encoding | lower range | upper range | 
| -- | -- | -- | -- | -- | -- | -- | 
| 11 |  uint8 | integer | 1 | none | 0 | 255 |  
| 12 |  unit16 | integer | 2 | none | 0 | 65535 | 
| 13 |  uint24 | integer | 3 | none | 0 | 16777215 |
| 14 |  uint32 | integer | 4 | none | 0 | 4294967295 | 
| 18 |  uint64 | integer | 8 | none | 0 |  18446744073709551615 |
| 21 |  int8 | integer | 1 | twos compliment | -128 | 127 | 
| 22 |  int16 | integer | 2 | twos compliment | -32768 | 32767 | 
| 23 |  int24 | integer | 3 | twos compliment | -8388608 | 8388607 | 
| 24 |  int32 | integer | 4 | twos compliment | -2147483648 | 2147483647 | 
| 28 |  int64 | integer | 8 | twos compliment | -9223372036854775808 | 9223372036854775807 | 
| 32 |  float16 | half floating points | 2 | IEEE 754 (half) | -65504 | 65504 | 
| 34 |  float32 | single precsion float | 4 | IEEE 754 (single) | -3.40E+38 | 3.40E+38 | 
| 38 |  float64 | double precision float | 8 | IEEE 754 (double) | very little | very lots | 
| 41 |  1/2 point % | rounded positive decimal  | 1 | value / 0.5 | 0.0 | 127.5 | good for percentages to half a point. 
| 42 |  temps to 2dp 16 | rounded decimal | 2 |  = (value - 273.15) / 0.01 | -273.15 | 382.20 | "useful for environmental temperatures to 2 decimal places 
| but with no approximation that occurs in IEEE 754 
| and easier to quick arithmetic by eye"
| 52 |  small to 3dp 16 | rounded decimal | 2 |  = (value - 25 / 0.001 | -25.000 | 40.535 | 


This table can be expanded as required. 







