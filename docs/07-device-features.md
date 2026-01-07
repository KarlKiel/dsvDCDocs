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

```json
{
  "channelType": 1,
  "channelId": "brightness",
  "name": "Brightness",
  "min": 0.0,
  "max": 100.0,
  "resolution": 0.1,
  "writable": true
}
```

## Input Features

### Button Inputs

User interaction inputs (wall switches, remote buttons).

#### Button Types

| Type | Description |
|------|-------------|
| 0 | Undefined |
| 1 | Single button |
| 2 | Two-way rocker |
| 3 | Four-way button |

#### Button States

| State | Description |
|-------|-------------|
| 0 | Released |
| 1 | Pressed |
| 2 | Double-click |
| 3 | Long press |

#### Button Input Structure

```json
{
  "inputType": 1,
  "name": "Wall Button",
  "buttonType": 1,
  "buttonID": 0,
  "state": 0,
  "age": 0,
  "group": 1,
  "mode": 0
}
```

### Binary Inputs

On/off sensors (door/window contacts, motion sensors).

#### Binary Input Structure

```json
{
  "inputType": 2,
  "name": "Door Contact",
  "state": 0,  // 0=closed, 1=open
  "age": 0,
  "binInputType": 1  // Door/window
}
```

### Sensor Inputs

Analog/continuous value sensors.

#### Sensor Types

| Type | Name | Typical Range | Unit |
|------|------|---------------|------|
| 1 | Temperature | -40 to 80 | °C |
| 2 | Humidity | 0-100 | % |
| 3 | Brightness | 0-100000 | lux |
| 4 | CO2 | 0-2000 | ppm |
| 5 | Air Pressure | 900-1100 | hPa |
| 9 | Wind Speed | 0-50 | m/s |
| 10 | Power | 0-10000 | W |
| 11 | Energy | 0-100000 | kWh |

#### Sensor Input Structure

```json
{
  "inputType": 3,
  "name": "Temperature Sensor",
  "sensorType": 1,
  "sensorUsage": 0,  // Room temperature
  "state": 21.5,
  "age": 10,
  "min": -40.0,
  "max": 80.0,
  "resolution": 0.1,
  "updateInterval": 60
}
```

### Pushing Input State Changes

When input state changes, push notification:

```json
{
  "dSUID": "DEVICE_DSUID",
  "changedproperties": [
    {
      "name": "inputs",
      "elements": [
        {
          "name": "0",  // Input index
          "elements": [
            { "name": "state", "value": { "v_uint64": 1 } },
            { "name": "age", "value": { "v_uint64": 0 } }
          ]
        }
      ]
    }
  ]
}
```

## Scene Features

### Scene Storage

Devices store scene configurations:

```json
{
  "scenes": [
    {
      "sceneNo": 0,
      "name": "Off",
      "channels": [
        { "channelId": "brightness", "value": 0.0 }
      ]
    },
    {
      "sceneNo": 5,
      "name": "Scene 1",
      "channels": [
        { "channelId": "brightness", "value": 100.0 },
        { "channelId": "hue", "value": 180.0 }
      ]
    }
  ]
}
```

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

Some devices may have dynamic channels:

```json
{
  "output": {
    "channels": [
      {
        "channelId": "brightness",
        "name": "Brightness",
        "dynamic": false
      },
      {
        "channelId": "effect",
        "name": "Current Effect",
        "dynamic": true,
        "options": ["none", "colorloop", "strobe"]
      }
    ]
  }
}
```

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

- **Outputs**: Define channels in `/output/channels`
- **Inputs**: Define inputs in `/inputs`
- **Scenes**: Implement scene storage and recall
- **Actions**: Define in `/actions` array

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
