# Core Concepts

## Introduction

This document explores the fundamental concepts of the vDC API in greater depth, building on the introduction from [Getting Started](./01-getting-started.md).

## The digitalSTROM Ecosystem

### Physical digitalSTROM System

The traditional digitalSTROM system consists of:

1. **dSS (digitalSTROM Server)**: Central brain of the system
2. **dSM (digitalSTROM Meter)**: Hardware modules that communicate over powerline
3. **dSD (digitalSTROM Devices)**: Physical devices (switches, lights, sensors)

### Virtual Integration Layer

The vDC API extends this with virtual components:

1. **vdSM (Virtual dS Meter)**: Software component in dSS that connects to external devices
2. **vDC Host**: Your software that bridges external devices to digitalSTROM
3. **vDC (Virtual Device Connector)**: Logical grouping of similar devices
4. **vdSD (Virtual dS Device)**: Individual external devices

## Understanding vDC vs vDC Host

### vDC Host

The **vDC host** is the software you implement:

- Runs as a network service
- Implements the vDC protocol
- Manages TCP connections
- Communicates with external devices/systems
- Can host multiple vDCs

**Analogy**: The vDC host is like an embassy that represents multiple countries (vDCs).

### vDC (Virtual Device Connector)

A **vDC** is a logical entity representing a device technology class:

- One per device technology/brand
- Has its own unique dSUID
- Groups related devices
- Defines capabilities

**Examples**:
- Philips Hue vDC (manages all Hue devices)
- Sonos vDC (manages all Sonos speakers)
- MQTT vDC (manages MQTT-connected devices)

**Analogy**: A vDC is like a department within an embassy.

### Relationship Hierarchy

```
vDC Host (Your Software)
├── vDC #1 (Philips Hue)
│   ├── vdSD (Hue Bulb 1)
│   ├── vdSD (Hue Bulb 2)
│   └── vdSD (Hue Strip 1)
├── vDC #2 (Sonos)
│   ├── vdSD (Living Room Speaker)
│   └── vdSD (Kitchen Speaker)
└── vDC #3 (MQTT Sensors)
    ├── vdSD (Temperature Sensor)
    └── vdSD (Motion Sensor)
```

## Device Types and Classifications

### Primary Groups (Functional Colors)

digitalSTROM uses color-coded functional groups:

| Group | Color | Typical Devices |
|-------|-------|-----------------|
| 1 | Yellow | Light |
| 2 | Gray | Blinds/Shades |
| 3 | Blue | Climate (Heating/Cooling) |
| 4 | Cyan | Audio |
| 5 | Magenta | Video |
| 8 | Black | Joker (Multiple functions) |
| 9 | White | Single devices |
| 48 | Green | Access/Security |

**Usage**: Devices declare their `primaryGroup` to indicate function.

### Device Classes

Devices can have various capabilities:

**Output Devices**:
- Lights (dimmable, color)
- Switches (on/off)
- Blinds (position control)
- Audio (volume, source)
- Climate (temperature setpoint)

**Input Devices**:
- Buttons (wall switches)
- Sensors (temperature, humidity, motion)
- Binary inputs (door/window contacts)

**Hybrid Devices**:
- Can have both inputs and outputs
- Example: Smart switch with physical button

## dSUID - Unique Identification

### What is a dSUID?

A **dSUID** (digitalSTROM Unique IDentifier) is a unique identifier for every entity in the system:

- 17-byte (136-bit) identifier
- Represented as hex string (34 characters)
- Must be globally unique
- Remains stable across reboots

### dSUID Format

```
dSUID format: [16 bytes device-specific] [1 byte subdevice]
```

**Example**: `0001020304050607080910111213141500`

### Generating dSUIDs

For virtual devices, use **deterministic generation**:

**Method 1: UUID v5 (Recommended)**
```python
import uuid

# Define a namespace for your vDC
VDC_NAMESPACE = uuid.UUID('12345678-1234-5678-1234-567812345678')

# Generate dSUID from device's unique identifier
device_id = "hue-bulb-12345"
device_uuid = uuid.uuid5(VDC_NAMESPACE, device_id)
dsuid = device_uuid.bytes + b'\x00'  # Add subdevice byte
dsuid_hex = dsuid.hex().upper()
```

**Method 2: From MAC Address**
```python
# For devices with MAC addresses
mac = "AA:BB:CC:DD:EE:FF"
# Derive dSUID incorporating MAC
```

**Requirements**:
- Must be deterministic (same input = same dSUID)
- Must be unique across all devices
- Should remain stable (same device = same dSUID)

### dSUID Best Practices

1. **Namespace**: Use a unique namespace for your vDC implementation
2. **Stability**: Keep dSUIDs stable across restarts
3. **Uniqueness**: Ensure global uniqueness
4. **Derivation**: Base on stable device identifiers (serial numbers, MAC, etc.)

## Connection Management

### Discovery

vDC hosts advertise themselves using **Avahi/mDNS** (Bonjour):

**Service Type**: `_ds-vdc._tcp`

**Example Avahi Service File** (`/etc/avahi/services/vdc.service`):
```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name>My vDC Host</name>
  <service>
    <type>_ds-vdc._tcp</type>
    <port>8440</port>
    <txt-record>vdchost=my-vdc-host</txt-record>
  </service>
</service-group>
```

**TXT Records** (optional):
- `vdchost`: Human-readable name
- `model`: vDC host model
- Additional metadata

### Connection Sequence

1. **vdSM discovers** vDC host via mDNS
2. **vdSM connects** to vDC host TCP port
3. **Handshake** (Hello exchange)
4. **vDC announcements** (one per vDC)
5. **Device announcements** (one per device)
6. **Operational** (property access, notifications)
7. **Keep-alive** (ping/pong)

### Connection States

**vDC Host Perspective**:
```
IDLE → CONNECTED → HELLO_RECEIVED → READY → OPERATIONAL
                ↓                                    ↓
              CLOSED ← CLOSING ← BYE_RECEIVED ←────┘
```

**States**:
- **IDLE**: No connection
- **CONNECTED**: TCP connected, awaiting hello
- **HELLO_RECEIVED**: Handshake complete
- **READY**: vDCs and devices announced
- **OPERATIONAL**: Normal operation
- **CLOSING**: Shutdown in progress
- **CLOSED**: Connection terminated

### Reconnection Strategy

**On Connection Loss**:

1. Clean up session state
2. Close TCP socket
3. Wait for new vdSM connection
4. Or attempt reconnection (if host initiates)

**On Restart**:

1. Re-announce all vDCs
2. Re-announce all devices
3. vdSM will query properties again

## Zones and Groups

### Zones

Zones represent physical locations:

- Room-level granularity
- Each device assigned to one zone
- Zone ID is a number (0-65535)
- Configure via `zoneID` property

**Common Zones**:
- 1: Living Room
- 2: Kitchen
- 3: Bedroom
- etc.

### Groups

Groups are functional classifications:

- Based on device type/capability
- Color-coded (yellow, gray, blue, etc.)
- Defined by `primaryGroup` property
- Used for scene routing

**Usage**:
- Scenes are group-specific
- Notifications can filter by group
- UI organizes by groups

## Scenes

### Scene Concept

Scenes are pre-defined states for devices:

- Numbered 0-127
- Group-specific (lights, blinds, etc.)
- Store device states (brightness, position, etc.)
- Recalled by scene number

### Standard Scenes

**Common Scene Numbers**:

| Number | Name | Typical Behavior |
|--------|------|------------------|
| 0 | Off | Turn off |
| 5 | Scene 1 / Preset 1 | User-defined |
| 17 | Scene 2 / Preset 2 | User-defined |
| 18 | Scene 3 / Preset 3 | User-defined |
| 19 | Scene 4 / Preset 4 | User-defined |
| 14 | Auto Off | Automatic off |
| 13 | Standby | Low power mode |

### Scene Operations

**Call Scene**: Activate scene
```protobuf
vdsm_NotificationCallScene {
    dSUID: ["device1", "device2"]
    scene: 5  // Scene 1
}
```

**Save Scene**: Store current state
```protobuf
vdsm_NotificationSaveScene {
    dSUID: ["device1"]
    scene: 5
}
```

**Undo Scene**: Revert to previous
```protobuf
vdsm_NotificationUndoScene {
    dSUID: ["device1"]
    scene: 5
}
```

### Scene Storage

vDC hosts should:
- Store scene values persistently
- Restore on restart
- Provide default values for unconfigured scenes

## Inputs and Outputs

### Output Channels

Output channels represent controllable aspects:

**Light Device**:
- Channel 0: Brightness (0-100%)
- Channel 1: Hue (0-360°)
- Channel 2: Saturation (0-100%)

**Blind Device**:
- Channel 0: Position (0-100%)
- Channel 1: Angle/Slat (0-100%)

### Channel Operations

**Set Value**:
```protobuf
vdsm_NotificationSetOutputChannelValue {
    dSUID: ["device1"]
    channel: 0      // Channel index (0 = default channel)
    value: 75.0
}
```

**Dim** (gradual change):
```protobuf
vdsm_NotificationDimChannel {
    dSUID: ["device1"]
    channel: 0      // Channel index (0 = default channel)
    mode: 1         // Start dimming
    area: 1         // Direction: up
}
```

### Input Types

**Button Inputs**:
- User interaction (press, release)
- Generate state changes
- Multiple button types (single, double, long press)

**Sensor Inputs**:
- Continuous values (temperature, humidity)
- Push updates when changed
- Have resolution and units

**Binary Inputs**:
- On/off states (door open/closed)
- Contact sensors
- Occupancy detectors

## Capabilities

### vDC Capabilities

vDCs declare capabilities:

```json
{
  "capabilities": {
    "metering": false,        // Power measurement
    "identification": true,   // Can identify hardware
    "dynamicDefinitions": false
  }
}
```

### Device Capabilities

Devices implicitly declare capabilities through properties:

- **Has outputs**: Can be controlled
- **Has inputs**: Can generate events
- **Has scenes**: Can recall scenes
- **Has sensors**: Provides sensor values

## State Management

### Device State

Devices maintain state:

- Current output values
- Input states
- Scene configurations
- Settings/configuration

### State Synchronization

**Push Model** (Recommended):
- Device state changes → Push notification to vdSM
- Fast, efficient
- Requires reliable push mechanism

**Pull Model**:
- vdSM queries state periodically
- Simple but inefficient
- Fallback if push unavailable

### State Persistence

vDC hosts should persist:
- Device configurations
- Scene values
- User settings
- NOT: Transient states (unless needed for restore)

## Error Handling and Resilience

### Connection Resilience

**Handle Connection Loss**:
```python
def on_connection_lost():
    # Clean up resources
    # Mark devices as unavailable
    # Wait for reconnection
    # Re-announce on reconnection
```

**Handle Protocol Errors**:
```python
def on_invalid_message(message):
    # Log error
    # Return appropriate error code
    # Don't crash
```

### Device Resilience

**Device Offline**:
- Mark as unavailable in property tree
- Return ERR_SERVICE_NOT_AVAILABLE for operations
- Re-announce when back online

**External API Errors**:
- Translate to appropriate vDC error codes
- Provide meaningful error descriptions
- Retry with backoff if transient

## Performance Considerations

### Property Access Optimization

- **Cache frequent properties**: name, dSUID, model
- **Lazy load**: Don't query external device unless needed
- **Batch updates**: Combine multiple property changes

### Message Batching

- Combine multiple property updates in one PushNotification
- Don't send individual updates for each channel

### Connection Efficiency

- Reuse TCP connection
- Implement proper keep-alive
- Don't create unnecessary connections

## Security Considerations

### Network Security

- **Limit Access**: Bind to specific interfaces
- **Firewall**: Restrict connections to trusted networks
- **TLS**: Consider encryption for sensitive deployments

### Input Validation

- **Validate all inputs**: Check message structure
- **Sanitize dSUIDs**: Validate format
- **Range check**: Verify value ranges
- **Property existence**: Check before access

### Resource Limits

- **Connection limit**: Prevent DoS
- **Message size limit**: Prevent memory exhaustion
- **Rate limiting**: Prevent flooding

## Summary

Key concepts to remember:

1. **Hierarchy**: vDC Host → vDC → vdSD
2. **dSUID**: Unique, stable identifiers for all entities
3. **Discovery**: Avahi/mDNS for automatic detection
4. **Property Trees**: Unified access to all device aspects
5. **Scenes**: Pre-defined device states
6. **Channels**: Controllable device outputs
7. **Groups**: Functional classification (colors)
8. **Zones**: Physical location grouping

## Next Steps

- [Protocol Basics](./03-protocol-basics.md) - Detailed protocol specification
- [Property System](./04-property-system.md) - Deep dive into properties
- [Implementation Guide](./10-implementation-guide.md) - Step-by-step implementation

---

[← Back: Getting Started](./01-getting-started.md) | [Back to Index](./README.md) | [Next: Protocol Basics →](./03-protocol-basics.md)
