# Virtual Device (vdSD) Reference

## Overview

This document provides detailed information about virtual digitalSTROM devices (vdSD), their properties, lifecycle, and operations.

For complete property system documentation, see [Property System](./04-property-system.md).

## Device Properties

### Essential Properties

All devices must provide these core properties (common properties from Section 2 of original API):

| Property Path | Type | R/W | Description |
|---------------|------|-----|-------------|
| `/name` | string | R/W | Device name (user-configurable) |
| `/dSUID` | string | R | Unique device identifier (34 hex characters) |
| `/displayId` | string | R | Human-readable ID printed on device (optional) |
| `/type` | string | R | Always "vdSD" for devices |
| `/model` | string | R | Device model name |
| `/modelVersion` | string | R | Model version |
| `/modelUID` | string | R | Model unique identifier (functional model) |
| `/hardwareVersion` | string | R | Hardware version (optional) |
| `/hardwareGuid` | string | R | Hardware GUID in URN format (optional) |
| `/hardwareModelGuid` | string | R | Hardware model GUID (optional) |
| `/vendorName` | string | R | Device vendor (optional) |
| `/vendorGuid` | string | R | Vendor GUID in URN format (optional) |
| `/oemGuid` | string | R | OEM product GUID (optional) |
| `/oemModelGuid` | string | R | OEM model GUID/GTIN (optional) |
| `/configURL` | string | R | Configuration URL (optional) |
| `/deviceIcon16` | binary | R | 16x16 pixel PNG icon (optional) |
| `/deviceIconName` | string | R | Icon filename-safe name (optional) |
| `/deviceClass` | string | R | Device class profile name (optional) |
| `/deviceClassVersion` | string | R | Device class profile version (optional) |
| `/active` | bool | R | Operation state of device (optional) |
| `/primaryGroup` | uint | R | Primary functional group (color) |
| `/zoneID` | uint | R/W | Zone assignment |

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
    { "name": "outputDescription" },
    { "name": "channelDescriptions" },
    { "name": "buttonInputDescriptions" },
    { "name": "sensorDescriptions" }
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
- `/outputDescription` - Output description
- `/channelDescriptions` - Array of channel descriptors
- `/channelStates` - Array of channel states

### Input Devices

Devices with inputs (buttons, sensors).

**Required properties** (one or more of):
- `/buttonInputDescriptions`, `/buttonInputSettings`, `/buttonInputStates` - For button inputs
- `/binaryInputDescriptions`, `/binaryInputSettings`, `/binaryInputStates` - For binary inputs
- `/sensorDescriptions`, `/sensorSettings`, `/sensorStates` - For sensor inputs

### Hybrid Devices

Devices with both inputs and outputs.

**Example**: Smart switch with dimmer and physical button

## Output Channels

Devices with outputs have several related property groups:

### Output Description

The `/outputDescription` container provides invariable output properties:

| Property | Type | Description |
|----------|------|-------------|
| `/outputDescription/function` | string | Output function (e.g., "light", "shadow") |
| `/outputDescription/outputUsage` | uint | Output usage type |

### Channel Descriptions

The `/channelDescriptions` array contains invariable properties for each channel:

| Property | Type | Description |
|----------|------|-------------|
| `/channelDescriptions/N/name` | string | Human-readable channel name |
| `/channelDescriptions/N/channelType` | uint | Channel type (see table below) |
| `/channelDescriptions/N/dsIndex` | uint | Index 0..N-1 (index 0 is default channel) |
| `/channelDescriptions/N/min` | double | Minimum value |
| `/channelDescriptions/N/max` | double | Maximum value |
| `/channelDescriptions/N/resolution` | double | Resolution/step size |

### Channel Settings

The `/channelSettings` array contains per-channel settings (currently no standard settings defined).

### Channel States

The `/channelStates` array contains current channel values (read-only, use setOutputChannelValue to change):

| Property | Type | Description |
|----------|------|-------------|
| `/channelStates/N/value` | double | Current channel value |
| `/channelStates/N/age` | double | Age in seconds (NULL if not yet applied) |

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
  "outputDescription": {
    "function": "light",
    "outputUsage": 0
  },
  "channelDescriptions": [
    {
      "name": "Brightness",
      "channelType": 1,
      "dsIndex": 0,
      "min": 0.0,
      "max": 100.0,
      "resolution": 0.1
    },
    {
      "name": "Hue",
      "channelType": 2,
      "dsIndex": 1,
      "min": 0.0,
      "max": 360.0,
      "resolution": 1.0
    }
  ],
  "channelStates": [
    {
      "value": 0.0,
      "age": 0.0
    },
    {
      "value": 0.0,
      "age": 0.0
    }
  ]
}
```

## Input Definitions

Devices with inputs use separate property groups for each input type:

### Button Inputs

Button inputs use three property arrays:

**`/buttonInputDescriptions`** - Invariable properties:
- `/buttonInputDescriptions/N/name` - Button name
- `/buttonInputDescriptions/N/dsIndex` - Index 0..N-1
- `/buttonInputDescriptions/N/supportsLocalKeyMode` - Can be local button
- `/buttonInputDescriptions/N/buttonID` - Physical button ID (optional)
- `/buttonInputDescriptions/N/buttonType` - Type (0=undefined, 1=single, 2=2-way, etc.)
- `/buttonInputDescriptions/N/buttonElementID` - Element of multi-contact button

**`/buttonInputSettings`** - Configurable settings:
- `/buttonInputSettings/N/group` - dS group number
- `/buttonInputSettings/N/function` - Button function (0=device, 5=room, etc.)
- `/buttonInputSettings/N/mode` - Mode (0=standard, 2=presence, 255=inactive, etc.)
- `/buttonInputSettings/N/channel` - Channel to control
- `/buttonInputSettings/N/setsLocalPriority` - Sets local priority flag
- `/buttonInputSettings/N/callsPresent` - Calls present if absent

**`/buttonInputStates`** - Current state:
- `/buttonInputStates/N/value` - Boolean state or NULL (pressed/released)
- `/buttonInputStates/N/clickType` - Click type (0=tip_1x, 1=tip_2x, 4=hold_start, 255=idle, etc.)
- `/buttonInputStates/N/age` - Age in seconds (or NULL)
- `/buttonInputStates/N/error` - Error state (0=ok, 1=open circuit, etc.)

### Binary Inputs

Binary inputs use three property arrays:

**`/binaryInputDescriptions`** - Invariable properties:
- `/binaryInputDescriptions/N/name` - Input name
- `/binaryInputDescriptions/N/dsIndex` - Index 0..N-1
- `/binaryInputDescriptions/N/inputType` - 0=poll only, 1=detects changes
- `/binaryInputDescriptions/N/inputUsage` - Usage type enum

**`/binaryInputSettings`** - Configurable settings (similar to button settings)

**`/binaryInputStates`** - Current state:
- `/binaryInputStates/N/value` - Boolean state or NULL
- `/binaryInputStates/N/extendedValue` - Extended value (replaces value if present)
- `/binaryInputStates/N/age` - Age in seconds
- `/binaryInputStates/N/error` - Error state

### Sensor Inputs

Sensor inputs use three property arrays:

**`/sensorDescriptions`** - Invariable properties:
- `/sensorDescriptions/N/name` - Sensor name
- `/sensorDescriptions/N/dsIndex` - Index 0..N-1
- `/sensorDescriptions/N/sensorType` - Type (1=temperature, 2=humidity, 3=illumination, etc.)
- `/sensorDescriptions/N/sensorUsage` - Usage type
- `/sensorDescriptions/N/min` - Minimum value
- `/sensorDescriptions/N/max` - Maximum value
- `/sensorDescriptions/N/resolution` - Resolution
- `/sensorDescriptions/N/updateInterval` - Update interval in seconds
- `/sensorDescriptions/N/aliveSignInterval` - Alive sign interval in seconds

**`/sensorSettings`** - Configurable settings

**`/sensorStates`** - Current readings:
- `/sensorStates/N/value` - Sensor value (double or NULL)
- `/sensorStates/N/age` - Age in seconds
- `/sensorStates/N/error` - Error state

### Example Input Definitions

**Button Input Example**:
```json
{
  "buttonInputDescriptions": [
    {
      "name": "Wall Button",
      "dsIndex": 0,
      "supportsLocalKeyMode": true,
      "buttonID": 0,
      "buttonType": 1,
      "buttonElementID": 0
    }
  ],
  "buttonInputSettings": [
    {
      "group": 1,
      "function": 0,
      "mode": 0,
      "channel": 0,
      "setsLocalPriority": false,
      "callsPresent": false
    }
  ],
  "buttonInputStates": [
    {
      "value": false,
      "clickType": 255,
      "age": 0.0,
      "error": 0
    }
  ]
}
```

**Sensor Input Example**:
```json
{
  "sensorDescriptions": [
    {
      "name": "Temperature",
      "dsIndex": 0,
      "sensorType": 1,
      "sensorUsage": 0,
      "min": -40.0,
      "max": 80.0,
      "resolution": 0.1,
      "updateInterval": 60,
      "aliveSignInterval": 300
    }
  ],
  "sensorStates": [
    {
      "value": 21.5,
      "age": 10.0,
      "error": 0
    }
  ]
}
```

## Scenes

Devices support scenes for storing and recalling states.

### Scene Configuration

The `/scenes` array contains scene configurations. Each scene element has:

| Property | Type | Description |
|----------|------|-------------|
| `/scenes/N` | container | Scene N (named by scene number) |
| `/scenes/N/channels` | array | Channel values for this scene |
| `/scenes/N/channels/M/value` | double | Value for channel M |
| `/scenes/N/channels/M/dontCare` | bool | If true, don't apply this channel value |
| `/scenes/N/channels/M/automatic` | bool | If true, activate automatic control |

**Note**: Scene elements are named by scene number (e.g., "0", "5", "17"), not sequential indices.

Stored in `/scenes` array:

```json
{
  "scenes": [
    {
      "name": "0",  // Scene number as string
      "channels": [
        {
          "name": "0",  // Channel index
          "value": 0.0,
          "dontCare": false,
          "automatic": false
        }
      ]
    },
    {
      "name": "5",  // Scene 5
      "channels": [
        {
          "name": "0",
          "value": 100.0,
          "dontCare": false,
          "automatic": false
        }
      ]
    },
    {
      "name": "17",  // Scene 17
      "channels": [
        {
          "name": "0",
          "value": 30.0,
          "dontCare": false,
          "automatic": false
        }
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
    channel: 0       // Channel index (0 = default channel)
    value: 75.0
    apply_now: true
}
```

### Dim Channel

```protobuf
vdsm_NotificationDimChannel {
    dSUID: ["DEVICE_DSUID"]
    channel: 0       // Channel index (0 = default channel)
    mode: 1          // Start dimming
    area: 1          // Up
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
            name: "buttonInputStates"
            elements: [
                {
                    name: "0"
                    elements: [
                        { name: "value", value: { v_bool: true } },
                        { name: "clickType", value: { v_uint64: 0 } },
                        { name: "age", value: { v_double: 0.0 } }
                    ]
                }
            ]
        }
    ]
}
```

Or for sensor changes:

```protobuf
vdc_SendPushNotification {
    dSUID: "DEVICE_DSUID"
    changedproperties: [
        {
            name: "sensorStates"
            elements: [
                {
                    name: "0"
                    elements: [
                        { name: "value", value: { v_double: 22.3 } },
                        { name: "age", value: { v_double: 0.0 } }
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
