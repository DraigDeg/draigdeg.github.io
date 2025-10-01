---
sidebar_label: 'Wireless'
sidebar_position: 4
---

# Wireless Data Transmission Encoding Specification.

## Introduction.

The binary encoding for the wireless transmission of measurement data from the channels of modules follows a similar overall structure to the text-based encoding seen in the RA0 command, defined in the PROTOCOL.md document in this folder. 

The difference is that all descriptors, addresses and values are binary encoded into bytes for data transmission. 

Here follows a description that that byte-order encoding.

## Message Encoding. version 1

A transmitted message contains data from one or more modules.  Termed Module "Frames"
The data from a module contains one or more channels and each channel contains one more measurements. A measurement comprises a measurement type, an encoding type, and an encoded value. The value is binary encoded depending on the encoding type specified in that measurement. 

### Message Header

A message starts with the following header:

| bytes | length | data type | description |
| ----- | ------ | --------- | ----------- |
| 0     | 1      | uint8     | message version 0-255 - this section describes version 0 |
| 1     | 1      | uint8     | quantity of Module frames in this message 0-255 |

Following the message header is one or more module frame data payloads, the quantity of frames is defined in byte no.2.
The length of the message is dependant on these module frame and their contents. 

### Module Data Frame

A Module data frame defines the address of the module, and the channel data. 

A module data frame comprises the following:

| bytes | length | data type | description | 
| ----- | ------ | --------- | ----------- |
| 1     | 1      | uint8     | module slot number 1-4 |
| 2     | 1      | uint8     | module port number 1-4 |
| 3     | 1      | uint8     | quantity of channel data frames that follow in this module frame 1-255 |

Following the module frame header is one or more channel data frames, the quantity of which is defined in byte no.3 above.
The length of each module frame is dependant on these channel frames and their contents. 

### Channel Data Frame

A channel data frame defines the channel number and one or more measurement data frames

a channel data frame comprises the following:

| bytes | length | data type | description | 
| ----- | ------ | --------- | ----------- |
| 1     | 1      | uint8     | channel number 1-255 |
| 2     | 1      | uint8     | quantity of measurement data frames that follow in this channel frame 1-255 |

Following the channel frame header is one or more measurement data frames, the quantity of which is defined in byte no.2 above.
The length of each channel frame is dependant on these measurement frames and their contents. 

### Measurement Data Frame

A measurement data frame defines the measurement type, its encoding, and its value

a measurement data frame comprises the following:

| bytes | length | data type | description | 
| ----- | ------ | --------- | ----------- |
| 1     | 1      | uint8     | measurement type (see DATA_ENCODING.md) 1-255 |
| 2     | 1      | uint8     | encoding type (see DATA_ENCODING.md) 1-255 |
| 3...N | N      | \<encoded> | numeric value, encoded as per the above data encoding type in byte 2 of the measurement frame above. |



## The Things Network Decoder

```javascript
/**
 * The Things Network Payload Formatter for DraigDeg Wireless Messages
 * 
 * This decoder follows the binary format specified in WIRELESS.md
 * and is compatible with The Things Stack V3 payload formatters.
 * 
 * Usage: Copy this entire file into the "Formatter code" section
 * of your application's payload formatter in The Things Network console.
 */

// Encoding type constants
const ENCODING_UINT8 = 11;
const ENCODING_UINT16 = 12;
const ENCODING_UINT24 = 13;
const ENCODING_UINT32 = 14;
const ENCODING_UINT64 = 18;
const ENCODING_INT8 = 21;
const ENCODING_INT16 = 22;
const ENCODING_INT24 = 23;
const ENCODING_INT32 = 24;
const ENCODING_INT64 = 28;
const ENCODING_FLOAT16 = 32;
const ENCODING_FLOAT32 = 34;
const ENCODING_FLOAT64 = 38;
const ENCODING_HALF_PERCENT = 41;
const ENCODING_TEMP_2DP_16 = 42;
const ENCODING_SMALL_3DP_16 = 52;

// Measurement type names
const MEASUREMENT_TYPES = {
    1: "Temperature",
    2: "Humidity",
    3: "Voltage",
    4: "Current",
    5: "Digital"
};

// Measurement units
const MEASUREMENT_UNITS = {
    1: "°C",
    2: "%RH",
    3: "mV",
    4: "mA",
    5: ""
};

// Encoding type names
const ENCODING_NAMES = {
    [ENCODING_UINT8]: "uint8",
    [ENCODING_UINT16]: "uint16",
    [ENCODING_UINT24]: "uint24",
    [ENCODING_UINT32]: "uint32",
    [ENCODING_UINT64]: "uint64",
    [ENCODING_INT8]: "int8",
    [ENCODING_INT16]: "int16",
    [ENCODING_INT24]: "int24",
    [ENCODING_INT32]: "int32",
    [ENCODING_INT64]: "int64",
    [ENCODING_FLOAT16]: "float16",
    [ENCODING_FLOAT32]: "float32",
    [ENCODING_FLOAT64]: "float64",
    [ENCODING_HALF_PERCENT]: "1/2 point %",
    [ENCODING_TEMP_2DP_16]: "temps to 2dp 16",
    [ENCODING_SMALL_3DP_16]: "small to 3dp 16"
};

function getEncodingLength(encodingType) {
    const lengths = {
        [ENCODING_UINT8]: 1,
        [ENCODING_INT8]: 1,
        [ENCODING_HALF_PERCENT]: 1,
        [ENCODING_UINT16]: 2,
        [ENCODING_INT16]: 2,
        [ENCODING_FLOAT16]: 2,
        [ENCODING_TEMP_2DP_16]: 2,
        [ENCODING_SMALL_3DP_16]: 2,
        [ENCODING_UINT24]: 3,
        [ENCODING_INT24]: 3,
        [ENCODING_UINT32]: 4,
        [ENCODING_INT32]: 4,
        [ENCODING_FLOAT32]: 4,
        [ENCODING_UINT64]: 8,
        [ENCODING_INT64]: 8,
        [ENCODING_FLOAT64]: 8
    };
    return lengths[encodingType] || 0;
}

function decodeValue(bytes, offset, encodingType) {
    switch (encodingType) {
        case ENCODING_UINT8:
            return bytes[offset];
        
        case ENCODING_UINT16:
            return (bytes[offset] << 8) | bytes[offset + 1];
        
        case ENCODING_UINT24:
            return (bytes[offset] << 16) | (bytes[offset + 1] << 8) | bytes[offset + 2];
        
        case ENCODING_UINT32:
            return (bytes[offset] << 24) | (bytes[offset + 1] << 16) | 
                   (bytes[offset + 2] << 8) | bytes[offset + 3];
        
        case ENCODING_INT8:
            let val8 = bytes[offset];
            return val8 > 127 ? val8 - 256 : val8;
        
        case ENCODING_INT16:
            let val16 = (bytes[offset] << 8) | bytes[offset + 1];
            return val16 > 32767 ? val16 - 65536 : val16;
        
        case ENCODING_INT24:
            let val24 = (bytes[offset] << 16) | (bytes[offset + 1] << 8) | bytes[offset + 2];
            if (val24 & 0x800000) { // Sign extend
                val24 |= 0xFF000000;
                return val24 - 0x100000000;
            }
            return val24;
        
        case ENCODING_INT32:
            let val32 = (bytes[offset] << 24) | (bytes[offset + 1] << 16) | 
                        (bytes[offset + 2] << 8) | bytes[offset + 3];
            return val32 > 2147483647 ? val32 - 4294967296 : val32;
        
        case ENCODING_FLOAT32:
            // IEEE 754 float32 decoding
            let bits = (bytes[offset] << 24) | (bytes[offset + 1] << 16) | 
                       (bytes[offset + 2] << 8) | bytes[offset + 3];
            let sign = (bits >>> 31) === 0 ? 1.0 : -1.0;
            let exponent = (bits >>> 23) & 0xFF;
            let mantissa = (exponent === 0) ? 
                          (bits & 0x7FFFFF) << 1 : 
                          (bits & 0x7FFFFF) | 0x800000;
            
            if (exponent === 255) {
                return mantissa ? NaN : sign * Infinity;
            }
            
            let result = sign * mantissa * Math.pow(2, exponent - 150);
            return Math.round(result * 10000) / 10000; // Round to 4 decimal places
        
        case ENCODING_HALF_PERCENT:
            return bytes[offset] * 0.5;
        
        case ENCODING_TEMP_2DP_16:
            let tempVal = (bytes[offset] << 8) | bytes[offset + 1];
            if (tempVal > 32767) tempVal -= 65536;
            return (tempVal * 0.01) + 273.15;
        
        case ENCODING_SMALL_3DP_16:
            let smallVal = (bytes[offset] << 8) | bytes[offset + 1];
            if (smallVal > 32767) smallVal -= 65536;
            return (smallVal * 0.001) + 25.0;
        
        default:
            return 0;
    }
}

function decodeMeasurement(bytes, offset) {
    if (offset + 2 > bytes.length) {
        throw new Error("Incomplete measurement frame");
    }
    
    const measurementType = bytes[offset];
    const encodingType = bytes[offset + 1];
    offset += 2;
    
    const valueLength = getEncodingLength(encodingType);
    if (offset + valueLength > bytes.length) {
        throw new Error("Incomplete measurement value");
    }
    
    const value = decodeValue(bytes, offset, encodingType);
    
    const measurement = {
        type: measurementType,
        typeName: MEASUREMENT_TYPES[measurementType] || "Unknown",
        encoding: encodingType,
        encodingName: ENCODING_NAMES[encodingType] || "Unknown",
        value: value,
        unit: MEASUREMENT_UNITS[measurementType] || ""
    };
    
    // Special formatting for digital values
    if (measurementType === 5) {
        measurement.digitalState = value > 0 ? "ON" : "OFF";
    }
    
    return {
        measurement: measurement,
        newOffset: offset + valueLength
    };
}

function decodeChannel(bytes, offset) {
    if (offset + 2 > bytes.length) {
        throw new Error("Incomplete channel frame");
    }
    
    const channelNumber = bytes[offset];
    const measurementCount = bytes[offset + 1];
    offset += 2;
    
    const channel = {
        channelNumber: channelNumber,
        measurementCount: measurementCount,
        measurements: []
    };
    
    for (let i = 0; i < measurementCount; i++) {
        const result = decodeMeasurement(bytes, offset);
        channel.measurements.push(result.measurement);
        offset = result.newOffset;
    }
    
    return {
        channel: channel,
        newOffset: offset
    };
}

function decodeModule(bytes, offset) {
    if (offset + 3 > bytes.length) {
        throw new Error("Incomplete module frame");
    }
    
    const slot = bytes[offset];
    const port = bytes[offset + 1];
    const channelCount = bytes[offset + 2];
    offset += 3;
    
    const module = {
        slot: slot,
        port: port,
        channelCount: channelCount,
        channels: []
    };
    
    for (let i = 0; i < channelCount; i++) {
        const result = decodeChannel(bytes, offset);
        module.channels.push(result.channel);
        offset = result.newOffset;
    }
    
    return {
        module: module,
        newOffset: offset
    };
}

/**
 * Main decoder function for The Things Network
 * @param {Object} input - The input object from TTN
 * @param {Array} input.bytes - The LoRaWAN payload as byte array
 * @param {number} input.fPort - The LoRaWAN fPort
 * @returns {Object} Decoded payload object
 */
function decodeUplink(input) {
    const bytes = input.bytes;
    const fPort = input.fPort;
    
    try {
        if (!bytes || bytes.length < 2) {
            throw new Error("Message too short for header");
        }
        
        // Decode message header
        const version = bytes[0];
        const moduleCount = bytes[1];
        let offset = 2;
        
        const decoded = {
            version: version,
            moduleCount: moduleCount,
            modules: []
        };
        
        // Create flattened data for easier access
        const data = {};
        const warnings = [];
        
        // Decode modules
        for (let i = 0; i < moduleCount; i++) {
            const result = decodeModule(bytes, offset);
            decoded.modules.push(result.module);
            offset = result.newOffset;
            
            // Flatten data for easier access in TTN
            const module = result.module;
            const moduleKey = `module_${module.slot}_${module.port}`;
            
            module.channels.forEach(channel => {
                channel.measurements.forEach(measurement => {
                    const key = `${moduleKey}_ch${channel.channelNumber}_${measurement.typeName.toLowerCase()}`;
                    data[key] = measurement.value;
                    
                    // Add unit-specific keys
                    if (measurement.unit) {
                        data[`${key}_unit`] = measurement.unit;
                    }
                    
                    // Add digital state if applicable
                    if (measurement.digitalState) {
                        data[`${key}_state`] = measurement.digitalState;
                    }
                });
            });
        }
        
        if (offset !== bytes.length) {
            warnings.push(`Decoded ${offset} bytes but message contains ${bytes.length} bytes`);
        }
        
        return {
            data: {
                ...data,
                _raw: decoded,
                _version: version,
                _moduleCount: moduleCount
            },
            warnings: warnings.length > 0 ? warnings : undefined
        };
        
    } catch (error) {
        return {
            data: {},
            warnings: [],
            errors: [`Decoder error: ${error.message}`]
        };
    }
}

// For TTN V3 compatibility
if (typeof module !== 'undefined' && module.exports) {
    module.exports = { decodeUplink };
}

```