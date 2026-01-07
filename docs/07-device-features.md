# Device Features

## Overview

This document details the various features that devices can implement: inputs, outputs, scenes, actions, and states.

## Output Features

### Channels

Output channels represent controllable aspects of a device.

#### Common Channel Types

**Lighting Channels**:
- **Brightness** (channelType: 1): 0-100%
- **Hue** (channelType: 2): 0-360 degrees
- **Saturation** (channelType: 3): 0-100%
- **Color Temperature** (channelType: 4): 2000-8000 Kelvin

**Shade/Blind Channels**:
- **Position** (channelType: 10): 0-100%
- **Angle** (channelType: 11): 0-100%

**Audio Channels**:
- **Volume** (channelType: 20): 0-100%

#### Channel Definition Structure

Channels are defined in the `/channelDescriptions` array. Each channel element contains:

```json
{
  "name": "Brightness",
  "channelType": 1,
  "dsIndex": 0,
  "min": 0.0,
  "max": 100.0,
  "resolution": 0.1
}
```

**Note**: `channelId` (string) is NOT part of the original API specification. Channels are accessed by their `dsIndex`.

## Input Features

### Button Inputs

User interaction inputs (wall switches, remote buttons).

#### Button Types

| Type | Description |
|------|-------------|
| 0 | Undefined |
| 1 | Single pushbutton |
| 2 | Two-way pushbutton |
| 3 | Four-way navigation button |
| 4 | Four-way navigation with center button |
| 5 | Eight-way navigation with center button |
| 6 | On-off switch |

#### Click Types

| Type | Description |
|------|-------------|
| 0 | tip_1x (short single press) |
| 1 | tip_2x (double press) |
| 2 | tip_3x (triple press) |
| 3 | tip_4x (quadruple press) |
| 4 | hold_start (long press started) |
| 5 | hold_repeat (long press continuing) |
| 6 | hold_end (long press ended) |
| 7 | click_1x (confirmed single click) |
| 8 | click_2x (confirmed double click) |
| 9 | click_3x (confirmed triple click) |
| 10 | short_long |
| 11 | local_off |
| 12 | local_on |
| 13 | short_short_long |
| 14 | local_stop |
| 255 | idle (no recent click) |

#### Button Input Property Structure

Button inputs are represented using THREE separate property arrays:

**buttonInputDescriptions** (invariable):
```json
{
  "name": "Wall Button",
  "dsIndex": 0,
  "supportsLocalKeyMode": true,
  "buttonID": 0,
  "buttonType": 1,
  "buttonElementID": 0
}
```

**buttonInputSettings** (configurable):
```json
{
  "group": 1,
  "function": 0,
  "mode": 0,
  "channel": 0,
  "setsLocalPriority": false,
  "callsPresent": false
}
```

**buttonInputStates** (current state):
```json
{
  "value": false,
  "clickType": 255,
  "age": 0.0,
  "error": 0
}
```

### Binary Inputs

On/off sensors (door/window contacts, motion sensors).

#### Input Usage Types

| Type | Description |
|------|-------------|
| 0 | Undefined |
| 1 | Room temperature control |
| 2 | Presence detection |
| 3 | Light detection |
| 4 | Presence in darkness |
| 5 | Twilight detection |
| 6 | Motion detection |
| 7 | Motion in darkness |
| 8 | Smoke detection |
| 9 | Wind detection |
| 10 | Rain detection |
| 11 | Sun detection |
| 12 | Thermostat function |
| 13-17 | Window/door contact (various positions) |
| 18 | Window handle |
| 19 | Garage door |
| 20 | Sun protection |
| 21 | Frost |
| 22 | Heating activated |
| 23 | Service/maintenance required |

#### Binary Input Property Structure

Binary inputs use THREE separate property arrays:

**binaryInputDescriptions** (invariable):
```json
{
  "name": "Door Contact",
  "dsIndex": 0,
  "inputType": 1,
  "inputUsage": 13
}
```

**binaryInputSettings** (configurable):
```json
{
  "group": 1,
  "minPushInterval": 0,
  "changesOnlyInterval": 0
}
```

**binaryInputStates** (current state):
```json
{
  "value": false,
  "age": 0.0,
  "error": 0
}
```

### Sensor Inputs

Analog/continuous value sensors.

#### Sensor Types

| Type | Name | Unit |
|------|------|------|
| 0 | None | - |
| 1 | Temperature | °C |
| 2 | Relative humidity | % |
| 3 | Illumination | lux |
| 4 | Supply voltage level | V |
| 5 | CO concentration | ppm |
| 6 | Radon activity | Bq/m³ |
| 7 | Gas type sensor | - |
| 8 | Particles <10µm | μg/m³ |
| 9 | Particles <2.5µm | μg/m³ |
| 10 | Particles <1µm | μg/m³ |
| 11 | Room operating panel set point | % (0-100) |
| 12 | Fan speed | 0-1 (0=off, <0=auto) |
| 13 | Wind speed (average) | m/s |
| 14 | Active Power | W |
| 15 | Electric current | A |
| 16 | Energy Meter | kWh |
| 17 | Apparent Power | VA |
| 18 | Air pressure | hPa |
| 19 | Wind direction | degrees |
| 20 | Sound pressure level | dB |
| 21 | Precipitation intensity | mm/m² (last hour) |
| 22 | CO2 concentration | ppm |
| 23 | Wind gust speed | m/s |
| 24 | Wind gust direction | degrees |
| 25 | Generated Active Power | W |
| 26 | Generated Energy | kWh |
| 27 | Water Quantity | l |
| 28 | Water Flow Rate | l/s |

#### Sensor Input Property Structure

Sensor inputs use THREE separate property arrays:

**sensorDescriptions** (invariable):
```json
{
  "name": "Temperature Sensor",
  "dsIndex": 0,
  "sensorType": 1,
  "sensorUsage": 0,
  "min": -40.0,
  "max": 80.0,
  "resolution": 0.1,
  "updateInterval": 60,
  "aliveSignInterval": 300
}
```

**sensorSettings** (configurable):
```json
{
  "group": 1
}
```

**sensorStates** (current reading):
```json
{
  "value": 21.5,
  "age": 10.0,
  "error": 0
}
```

### Pushing Input State Changes

When input state changes, push notification using the specific property array:

**Button press notification**:
```json
{
  "dSUID": "DEVICE_DSUID",
  "changedproperties": [
    {
      "name": "buttonInputStates",
      "elements": [
        {
          "name": "0",
          "elements": [
            { "name": "value", "value": { "v_bool": true } },
            { "name": "clickType", "value": { "v_uint64": 0 } },
            { "name": "age", "value": { "v_double": 0.0 } }
          ]
        }
      ]
    }
  ]
}
```

**Sensor value change notification**:
```json
{
  "dSUID": "DEVICE_DSUID",
  "changedproperties": [
    {
      "name": "sensorStates",
      "elements": [
        {
          "name": "0",
          "elements": [
            { "name": "value", "value": { "v_double": 22.3 } },
            { "name": "age", "value": { "v_double": 0.0 } }
          ]
        }
      ]
    }
  ]
}
```

**Binary input change notification**:
```json
{
  "dSUID": "DEVICE_DSUID",
  "changedproperties": [
    {
      "name": "binaryInputStates",
      "elements": [
        {
          "name": "0",
          "elements": [
            { "name": "value", "value": { "v_bool": true } },
            { "name": "age", "value": { "v_double": 0.0 } }
          ]
        }
      ]
    }
  ]
}
```

## Scene Features

### Scene Storage

Devices store scene configurations in the `/scenes` property array. Scenes are named by scene number (as strings).

Each scene contains channel values:

```json
{
  "scenes": [
    {
      "name": "0",
      "channels": [
        {
          "name": "0",
          "value": 0.0,
          "dontCare": false,
          "automatic": false
        }
      ]
    },
    {
      "name": "5",
      "channels": [
        {
          "name": "0",
          "value": 100.0,
          "dontCare": false,
          "automatic": false
        }
      ]
    }
  ]
}
```

**Scene properties per channel**:
- **value**: The channel value to apply when scene is called
- **dontCare**: If true, this channel value is not applied
- **automatic**: If true, activates automatic control logic

### Scene Operations

**Call Scene**: Apply scene configuration to device  
**Save Scene**: Store current channel values as scene  
**Undo Scene**: Revert to previous state before scene was called

See [API Operations](./08-api-operations.md) for implementation details.

## Action Features

Actions are named operations devices can perform.

### Standard Actions

Common actions available on devices:

- **identify**: Make device identify itself (blink, beep)
- **reset**: Reset device to defaults
- **calibrate**: Calibrate device (e.g., blind end positions)

### Custom Actions

Devices can define custom actions:

```json
{
  "actions": [
    {
      "actionId": "pulse",
      "name": "Pulse",
      "description": "Pulse the light briefly",
      "params": [
        {
          "name": "duration",
          "type": "numeric",
          "min": 0.1,
          "max": 5.0,
          "default": 1.0
        }
      ]
    }
  ]
}
```

### Invoking Actions

Use generic request (API v2c+):

```protobuf
vdsm_RequestGenericRequest {
    dSUID: "DEVICE_DSUID"
    methodname: "pulse"
    params: [
        { name: "duration", value: { v_double: 2.0 } }
    ]
}
```

## State Features

### Device States

Additional state information beyond channel values:

```json
{
  "states": [
    {
      "stateId": "connectivity",
      "name": "Connectivity",
      "value": "online",
      "type": "string"
    },
    {
      "stateId": "batteryLevel",
      "name": "Battery Level",
      "value": 85,
      "type": "numeric",
      "unit": "%"
    }
  ]
}
```

### Common State IDs

- **connectivity**: online/offline/error
- **batteryLevel**: 0-100%
- **signalStrength**: Signal quality
- **errorState**: Error description
- **operationMode**: Current mode

## Advanced Features

### Dynamic Channels

Some devices may have dynamic channel properties. The original API does not specify a standard "dynamic" flag or "options" field. These would be device-specific extensions if supported.

### Grouped Devices

Devices can act as groups:

```json
{
  "groupMembers": [
    "DEVICE1_DSUID",
    "DEVICE2_DSUID",
    "DEVICE3_DSUID"
  ]
}
```

Operations on the group affect all members.

## Implementation Guidelines

### Feature Detection

Implement only features your device supports:

- **Outputs**: Define in `/outputDescription`, `/channelDescriptions`, `/channelStates`
- **Button Inputs**: Define in `/buttonInputDescriptions`, `/buttonInputSettings`, `/buttonInputStates`
- **Binary Inputs**: Define in `/binaryInputDescriptions`, `/binaryInputSettings`, `/binaryInputStates`
- **Sensor Inputs**: Define in `/sensorDescriptions`, `/sensorSettings`, `/sensorStates`
- **Scenes**: Implement scene storage in `/scenes`
- **Actions**: Define in `/deviceActionDescriptions` and `/customActions`
- **States**: Define in `/deviceStateDescriptions` and `/deviceStates`

### State Management

- **Synchronize state**: Keep internal state in sync with physical device
- **Push updates**: Notify vdSM of state changes
- **Persist scenes**: Save scene configurations
- **Handle offline**: Gracefully handle device unavailability

### Performance

- **Batch updates**: Combine multiple channel changes
- **Debounce inputs**: Don't flood with button events
- **Cache values**: Cache frequently accessed properties
- **Lazy load**: Load details only when queried

## Related Documents

- [vdSD Reference](./06-vdsd-reference.md) - Device properties
- [API Operations](./08-api-operations.md) - Operation details
- [Property System](./04-property-system.md) - Property tree structure

---

[Back to Documentation Index](./README.md)
