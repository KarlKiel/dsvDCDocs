# Virtual Device (vdSD) Reference

## Overview

This document provides detailed information about virtual digitalSTROM devices (vdSD), their properties, lifecycle, and operations.

For complete property system documentation, see [Property System](./04-property-system.md).

## Device Properties

### Essential Properties

All devices must provide these core properties:

| Property Path | Type | R/W | Description |
|---------------|------|-----|-------------|
| `/name` | string | R/W | Device name (user-configurable) |
| `/dSUID` | string | R | Unique device identifier |
| `/model` | string | R | Device model name |
| `/modelVersion` | string | R | Model version |
| `/modelUID` | string | R | Model unique identifier |
| `/vendorName` | string | R | Device vendor |
| `/hardwareVersion` | string | R | Hardware version |
| `/serialNumber` | string | R | Serial number |
| `/softwareVersion` | string | R | Software/firmware version |
| `/primaryGroup` | uint | R | Primary functional group (color) |
| `/zoneID` | uint | R/W | Zone assignment |
| `/iconName` | string | R | Icon for UI display |

### Functional Group (primaryGroup)

Indicates device function/type:

| Value | Color | Function |
|-------|-------|----------|
| 1 | Yellow | Light |
| 2 | Gray | Blinds/Shades |
| 3 | Blue | Climate |
| 4 | Cyan | Audio |
| 5 | Magenta | Video |
| 8 | Black | Joker (multi-function) |
| 9 | White | Single devices |
| 48 | Green | Access/Security |

**Example**:
```json
{
  "name": "Living Room Light",
  "dSUID": "1234567890ABCDEF1234567890ABCDEF00",
  "model": "Hue Color Bulb",
  "modelVersion": "3.0",
  "vendorName": "Philips",
  "primaryGroup": 1,  // Yellow - Light
  "zoneID": 1,        // Living Room
  "iconName": "light_color"
}
```

## Device Lifecycle

### 1. Device Announcement

Announce device to vdSM:

```protobuf
vdc_SendAnnounceDevice {
    dSUID: "DEVICE_DSUID"
    vdc_dSUID: "PARENT_VDC_DSUID"
}
```

**When to announce**:
- After vDC announcement
- When new device discovered
- After device reconnection (if it vanished)

### 2. Initial Property Query

vdSM queries device properties:

**Common query**:
```json
{
  "query": [
    { "name": "name" },
    { "name": "model" },
    { "name": "primaryGroup" },
    { "name": "output" },
    { "name": "inputs" }
  ]
}
```

### 3. Operation

Normal device operation:
- Respond to property queries
- Handle notifications (scenes, channels)
- Push state changes

### 4. Device Removal

#### Temporary (Vanish)

Device going offline:

```protobuf
vdc_SendVanish {
    dSUID: "DEVICE_DSUID"
}
```

**When to send**:
- Device disconnected
- Device unreachable
- Temporary unavailability

**Re-announce** when device returns.

#### Permanent (Remove)

vdSM requests removal:

```protobuf
vdsm_SendRemove {
    dSUID: "DEVICE_DSUID"
}
```

**Action**: Clean up device resources, don't re-announce.

## Device Types

### Output Devices

Devices with controllable outputs (lights, blinds, etc.).

**Required properties**:
- `/output` container with channel definitions

### Input Devices

Devices with inputs (buttons, sensors).

**Required properties**:
- `/inputs` array with input definitions

### Hybrid Devices

Devices with both inputs and outputs.

**Example**: Smart switch with dimmer and physical button

## Output Channels

Devices with outputs must define channels in `/output/channels`:

### Channel Properties

| Property | Type | Description |
|----------|------|-------------|
| `channelType` | uint | Channel type (1=brightness, 2=hue, etc.) |
| `channelId` | string | Channel identifier (API v3+) |
| `name` | string | Human-readable name |
| `min` | double | Minimum value |
| `max` | double | Maximum value |
| `resolution` | double | Resolution/step size |

### Channel Types

| Type | Name | Range | Unit |
|------|------|-------|------|
| 1 | Brightness | 0-100 | % |
| 2 | Hue | 0-360 | degrees |
| 3 | Saturation | 0-100 | % |
| 4 | Color Temperature | 2000-8000 | Kelvin |
| 10 | Position | 0-100 | % |
| 11 | Angle | 0-100 | % |

### Example Output Definition

```json
{
  "output": {
    "channels": [
      {
        "channelType": 1,
        "channelId": "brightness",
        "name": "Brightness",
        "min": 0.0,
        "max": 100.0,
        "resolution": 0.1
      },
      {
        "channelType": 2,
        "channelId": "hue",
        "name": "Hue",
        "min": 0.0,
        "max": 360.0,
        "resolution": 1.0
      }
    ]
  }
}
```

## Input Definitions

Devices with inputs define them in `/inputs` array:

### Input Types

1. **Button Inputs**: User buttons/switches
2. **Binary Inputs**: On/off sensors (door contacts)
3. **Sensor Inputs**: Analog sensors (temperature, humidity)

### Input Properties

| Property | Type | Description |
|----------|------|-------------|
| `inputType` | uint | 1=button, 2=binary, 3=sensor |
| `name` | string | Input name |
| `state` | varies | Current state |
| `age` | uint | Seconds since last update |

### Example Input Definition

```json
{
  "inputs": [
    {
      "inputType": 1,
      "name": "Wall Button",
      "state": 0,
      "age": 0
    },
    {
      "inputType": 3,
      "name": "Temperature",
      "state": 21.5,
      "age": 10,
      "sensorType": 1,  // Temperature
      "sensorUsage": 0  // Room temperature
    }
  ]
}
```

## Scenes

Devices support scenes for storing and recalling states.

### Scene Configuration

Stored in `/scenes` array:

```json
{
  "scenes": [
    {
      "sceneNo": 5,
      "name": "Bright",
      "channels": [
        { "channelId": "brightness", "value": 100.0 }
      ]
    },
    {
      "sceneNo": 17,
      "name": "Dimmed",
      "channels": [
        { "channelId": "brightness", "value": 30.0 }
      ]
    }
  ]
}
```

### Scene Operations

See [API Operations](./08-api-operations.md) for details on:
- Call Scene
- Save Scene
- Undo Scene

## Device Operations

### Set Output Channel

```protobuf
vdsm_NotificationSetOutputChannelValue {
    dSUID: ["DEVICE_DSUID"]
    channelId: "brightness"
    value: 75.0
    apply_now: true
}
```

### Dim Channel

```protobuf
vdsm_NotificationDimChannel {
    dSUID: ["DEVICE_DSUID"]
    channelId: "brightness"
    mode: 1      // Start dimming
    area: 1      // Up
}
```

### Identify

Blink/beep to identify device:

```protobuf
vdsm_NotificationIdentify {
    dSUID: ["DEVICE_DSUID"]
}
```

## State Updates

### Push Notifications

Push state changes to vdSM:

```protobuf
vdc_SendPushNotification {
    dSUID: "DEVICE_DSUID"
    changedproperties: [
        {
            name: "inputs"
            elements: [
                {
                    name: "0"
                    elements: [
                        { name: "state", value: { v_uint64: 1 } }
                    ]
                }
            ]
        }
    ]
}
```

**When to push**:
- Button press/release
- Sensor value change
- Output value change (external control)
- State transitions

## Best Practices

### Device Naming

- Provide descriptive default names
- Allow user customization via `name` property
- Include location if known

### dSUID Generation

Generate stable, unique dSUIDs:

```python
import uuid

# Use device's unique ID (MAC, serial, etc.)
device_unique_id = "hue-bulb-12345"
namespace = uuid.UUID('your-vdc-namespace')
device_uuid = uuid.uuid5(namespace, device_unique_id)
device_dsuid = device_uuid.bytes + b'\x00'
```

### Scene Management

- Provide sensible default scenes
- Persist user-saved scenes
- Restore scenes on restart

### Error Handling

Return appropriate errors:

| Scenario | Error Code |
|----------|------------|
| Device offline | ERR_SERVICE_NOT_AVAILABLE |
| Invalid channel value | ERR_INVALID_VALUE_TYPE |
| Unsupported operation | ERR_NOT_IMPLEMENTED |
| Read-only property | ERR_FORBIDDEN |

## Related Documents

- [Device Features](./07-device-features.md) - Detailed feature descriptions
- [API Operations](./08-api-operations.md) - Operation details
- [Property System](./04-property-system.md) - Property tree fundamentals

---

[Back to Documentation Index](./README.md)
