---
sidebar_label: 'Module Protocol'
sidebar_position: 3
---

# Sensor Module Protocol

Request and Reply ASCII protocol to query and otherwise operate function modules

## Aims

* Scannable by human eyes for rapid development and diagnostics. 
* ASCII line oriented - actions performed upon receipt of line-feed "\n"

## Protocol Description

* Lines start with a line header
* Followed by line data - specific to the commands and their requests and responses
* Lines end with a line-feed "\n" (10)

### Numbers on the wire

Numbers are always encoded using ASCII printable characters to ensure readability.

E.g. 1 is expressed using the string "1" comprising the ASCII character decimal 49 (0x31)

E.g. 10 is expressed using the string "10" comprising two ASCII characters 0x31 and 0x30

E.g -10.2 is expressed using the string "-10.2" comprising five ASCII characters 

* `0x2D` for "-"
* `0x31` for "1"
* `0x30` for "0"
* `0x2E` for "."
* `0x32` for "2"

In the document we use the abbreviation to refer to ASCII numerals (and related signs, period chars etc used to express numbers)

### 1. Line header 

_Convention:  printable characters in double quotes, ascii decimal value in parenthesis_

1. Lines start with “?” (63) for controller initiated queries to modules
2. Lines start with “!” (33) for module responses back to the controller
3. The next characters are numerical digit chars "0-9" (48 - 57) which correspond to the numeric logical address of the module being queried, or the module responding
4. Followed by a comma “,” (44)

### 2. Line data
* Line data is specific to the command and responses detailed below
* Line data may only contain printable ASCII characters.
* Line data cannot contain a line feed “\n” (10) as this is the character that signifies the end of the the line. 
* A controller issues a single line command (see below)
E.g. “RA0” for Read-All (version 0)
* A module can issue zero-or-more interim (non-terminating) responses and then one single final terminating response. 

### Non-terminating Responses

#### Comment

1. “#” (35) for comments/debug 
2. Followed by a comma “,” (44)
3. Followed by ascii text that is ignored

**Example**

    !10,#,starting data gather

#### WAIT

1. “W” (87) for WAIT instruction
2. Followed by a comma “,” (44)
3. Followed by one or more ascii numeral digits "0-9" (48 - 57) to signify a delay in milliseconds:

**Example** 

    !10,W,750” to wait 750ms

### Terminating Responses

A module issues a single terminating response with either a command response (using the command request code) or an error

#### Command Response
1. Reply with the request command and version (E.g. “R0”)
2. Followed by a comma “,” (44)
3. Followed by response specific data

**Example** 

    !10,RA0,\<data>

#### Error Response    

1. Reply with “E” (69) for module error code if query is not possible or system fault
2. Followed by comma
3. Followed by numeric error code (see appendix)
4. Followed by comma
5. Followed by text

**Example** 

    !10,E,99,software error

### 3. Line End 

Lines are terminated with a line feed "\n" (10)


## Commands

### Ping - Device Check

Performs a simple request/reply - used for diagnostics.

**command code**

`PING,<msg>`

* "PING" ascii text
* followed by a comma “,” (44)
* followed by an ascii text message of any printable characters upto 100 chars long.

**response format**

`PING,PONG,<msg>`

* "PING" ascii text
* followed by a comma “,” (44)
* followed by the ascii text "PONG"
* followed by a comma “,” (44)
* followed by the ascii text message `<msg>` sent in the request. 



#### Example 1. 

The following full example sends a PING request to the module with logical address 3. 
Along with the module metadata, It responds with a single channel providing temperature {1} using encoding {42} and humidity {2} using encoding {41}

**Request**

    ?3,PING,test-test\n

**Response**

    !3,PING,PONG,test-test\n




### Describe All v0

Describe all measurements from all channels from a single module



**command code**

`DA0`

**response format**

* Manufacturers ID numeric (AN) 0-65535 (default 0 for self-built)
* Followed by a comma. “,” (44)
* Model, numeric, (AN) 0-65536
* Followed by a comma. “,” (44)
* hw revision, numeric, (AN) 0-255 
* Followed by a comma. “,” (44)
* Software revision, numeric, (AN) 0-255
* Followed by a comma. “,” (44)
* Module function bitmask numeric, (AN) 0-255,

  * bit 0 (provides measurements, exected on most modules)
  * bit 1 (provides datalogging)
  * bit 2 (requires main loop power, active for all slots)

THEN Repeating for each channel, concatenated, separated by comma ","

* Numeric Channel number (ascii numerals) E.g. "1"
* Followed by a comma. “,” (44)
* Followed by the number of measurements being supplied for this channel (ascii numerals) E.g. "2"

#### Example 1. 

The following full example requests the module and channel description from the module with logical address 3. 
Along with the module metadata, It responds with a single channel providing temperature {1} using encoding {42} and humidity {2} using encoding {41}

**Request**

    ?3,DA0\n

**Response**

    !3,DA0,0,0,1,5,1,2,1_42,2_41\n

| chars  | meaning |
| ------ | ------- |
|    `!` | response |
|    `3` | module 3 responding |
|  `DA0` | responding to command `DA0` |
|    `0` | manufacturers ID 0=self built |
|    `0` | manufacturers model code 0=self built |
|    `1` | hardware revision r1 |
|    `5` | software revision r5 |
|    `1` | Channel 1 data follows |
|    `2` | two measurements follow for channel 1 |
| `1_42` | temperature (1) should be encoded using format 42 |
| `2_41` | humidity {2} should be encoded using format {41} |


#### Example 2. 

The following full example requests the module and channel description from the module with logical address 3. 
Along with the module metadata, It responds with two channels, each providing a reading of millivolts {5} using encoding {22}


**Request**

    ?3,DA0\n

**Response**

    !3,DA0,1,1,5_22[3672],2,1,5_22[1256]\n


| chars  | meaning |
| ------ | ------- |
|    `!` | response |
|    `3` | module 3 responding |
|  `RA0` | responding to command `RA0` |
|  `DA0` | responding to command `DA0` |
|    `0` | manufacturers ID 0=self built |
|    `0` | manufacturers model code 0=self built |
|    `1` | hardware revision r1 |
|    `5` | software revision r5 |
|    `1` | Channel 1 data follows |
|    `1` | one measurement follows for channel 1 |
| `5_22` | voltage {5} should be encoded using format {22} |
|    `2` | Channel 2 data follows |
|    `1` | one measurement follows for channel 2 |
| `5_22` | voltage {5} should be encoded using format {22} |









### Read All v0

Read all measurements from all channels from a single module.

**command code**

`RA0`

**response format**

*Repeating for each channel, concatenated*

* Numeric Channel number (ascii numerals) E.g. "1"
* Followed by a comma. “,” (44)
* Followed by the number of measurements being supplied for this channel (ascii numerals) E.g. "2"
* Followed by repeating measurement descriptions and values as follows:

  * Measurement type code (AN) 0-255
  * followed by an underscore "_" (95)
  * Encoding type code (AN) 0-255
  * a square opening bracket "[" (91)
  * ASCII numerals (AN) value
  * a square closing bracket "]" (93)
  * followed by a comma

#### Example 1. 

The following full example requests all data from the module with logical address 3. It responds with a single channel providing temperature {1} using encoding {42} and humidity {2} using encoding {41}

**Request**

    ?3,RA0\n

**Response**

    !3,RA0,1,2,1_42[-10.2],2_41[67.5]\n

| chars  | meaning |
| ------ | ------- |
|    `!` | response |
|    `3` | module 3 responding |
|  `RA0` | responding to command `RA0` |
|    `1` | Channel 1 data follows |
|    `2` | two measurements follows |
| `1_42` | temperature (1) should be encoded using format 42 |
| `[-10.2]` | the temperature reading -10.2°C
| `2_41` | humidity {2} should be encoded using format {41} |
| `[67.5]` | the humidity reading |


#### Example 2. 

The following full example requests all data from the module with logical address 3. It responds with 2 channels, each providing a reading of millivolts {5} using encoding {22}


**Request**

    ?3,RA0\n

**Response**

    !3,RA0,1,1,5_22[3672],2,1,5_22[1256]\n

| chars  | meaning |
| ------ | ------- |
|    `!` | response |
|    `3` | module 3 responding |
|  `RA0` | responding to command `RA0` |
|    `1` | Channel 1 data follows |
|    `1` | one measurement follows |
| `5_22` | voltage {5} should be encoded using format {22} |
| `[3672]` | the millivolts reading 3672mV
|    `2` | Channel 2 data follows |
|    `1` | one measurement follows |
| `5_22` | voltage {5} should be encoded using format {22} |
| `[1256]` | the millivolts reading 1256mV |



