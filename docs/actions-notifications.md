# Actions and Notifications

## Overview

Actions and notifications enable device control and real-time communication in the vDC-API. This document describes all available actions and notifications for device operation.

## Actions vs Properties

### Actions
- Operations that may change device state and/or outputs
- Depend on preconditions  
- Do not represent a single distinct state value that can be read back
- Examples: Call scene, dim channel, identify

### Properties
- Distinct, unconditional state changes
- Can be read back
- Examples: Device name, zone assignment, configuration

> **Rule:** Use actions for operations, properties for state.

## Scene Operations

### Call Scene

Calls a scene on one or more devices.

**Notification:** `callScene`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationCallScene`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `scene` | integer | dS scene number to call |
| `force` | boolean | If true, local priority is overridden |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Notes
- `group` and `zone_id` are present when scene call applies to zone/group
- vdSM already creates separate calls for every involved device

### Save Scene

Saves the current device state into a scene.

**Notification:** `saveScene`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationSaveScene`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `scene` | integer | dS scene number to save state into |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### What is Saved
- Output value(s)
- Possibly multiple output values
- Device-specific state (flags, settings, etc.)

### Undo Scene

Undoes a previous scene call, restoring previous output values.

**Notification:** `undoScene`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationUndoScene`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `scene` | integer | Scene call to undo (must match last called scene) |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Behavior
- Restores output values to state before the scene call
- Only executes if device's last called scene matches `scene` parameter
- Prevents undoing a scene called from another source

### Call Minimum Scene

Sets device to minimum value to become logically "on" if currently off.

**Notification:** `callSceneMin`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationCallMinScene`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `scene` | integer | Scene to check for dontCare flag |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Behavior
- If dontCare flag is set: No action
- If dontCare flag is not set AND device is off: Set to minimum output level
- If device is already on: No action

### Set Local Priority

Sets device into local priority mode for area operations.

**Notification:** `setLocalPriority`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationSetLocalPrio`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `scene` | integer | Scene to check for dontCare flag |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Behavior
- Checks scene for dontCare flag
- If not set: Sets `localPriority` flag on device
- If set: No action

> **Note:** Compatibility method for dS 1.0 interfacing; may be removed in dS 2.x.

## Channel Operations

### Dim Channel

Performs dimming on a specific device channel.

**Notification:** `dimChannel`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationDimChannel` (API v2+)

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `channel` | integer | Channel to dim (0=default, 1-239=specific channel type) |
| `channelId` | string (optional) | Channel ID string (API v3+) |
| `mode` | integer | 1=dim up, -1=dim down, 0=stop dimming |
| `area` | integer | 0=no restriction, 1-4=area restriction by scene flag |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Channel Values
- **0:** Default channel (e.g., brightness for lights)
- **1-239:** Channel of specified type, if available
- Device ignores if channel type not available

#### Mode Values
- **1:** Start dimming upwards (increment output value)
- **-1:** Start dimming downwards (decrement output value)
- **0:** Stop dimming

#### Area Values
- **0:** No area restriction
- **1-4:** Only perform if corresponding area on scene (Tx_S1) doesn't have dontCare flag set

### Set Output Channel Value

Sets the value of an output channel.

**Notification:** `setOutputChannelValue`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationSetOutputChannelValue`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `channel` | integer | Channel to set (0=default, 1-239=specific) |
| `channelId` | string (optional) | Channel ID string (API v3+) |
| `value` | double | Value to apply (may be omitted if `automatic` is set) |
| `apply_now` | boolean (optional) | If false, buffer value; if true/omitted, apply all buffered values |
| `automatic` | boolean (optional) | Switch device to internal automatic control logic |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Buffering Behavior
- `apply_now=false`: New value buffered but not applied to hardware
- `apply_now=true` or omitted: All buffered values applied immediately

### Set Control Value

Sets the value of a control (sensor-like input affecting outputs).

**Notification:** `setControlValue`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationSetControlValue`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `name` | string | Control value name (defines semantic meaning) |
| `value` | double | Control value (dS sensor value) |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Control Names
- Maps to dS "sensor type" numbers in dS 1.0
- Defines how value affects device output(s)
- See "Control Values" chapter in vDC API properties specification

## Device Identification

### Identify

Makes device physically identifiable to user.

**Notification:** `identify`

**Direction:** vdSM → vDC host

**Message:** `vdsm_NotificationIdentify`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string[] (34 hex each) | One or multiple dSUIDs of target device(s) |
| `group` | integer (optional) | dS group number (informational only) |
| `zone_id` | integer (optional) | dS global zone ID (informational only) |

#### Implementation Examples
- **Lights:** Blink the controlled light
- **LED Indicators:** Flash indicator LED
- **Audio Devices:** Beep or hum
- **Mechanical Devices:** Short movement

## Generic Device Actions

### Call Action (API v2c+)

Calls a generic action on the device.

**Method:** `GenericRequest`

**Direction:** vdSM → vDC host

**Message:** `vdsm_RequestGenericRequest`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `methodname` | string | "invokeDeviceAction" |
| `dSUID` | string (34 hex) | dSUID of target device |
| `params` | PropertyElement[] | Additional parameters:<br>- `id` (string): device action to execute<br>- additional action-specific params |

#### Example Actions
Specific actions depend on device capabilities. Check device action descriptions.

## Configuration and Management

### Learn-In and Learn-Out

Starts the learn-in or learn-out process for devices.

**Method:** `GenericRequest`

**Direction:** vdSM → vDC host

**Message:** `vdsm_RequestGenericRequest`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `methodname` | string | "pair" |
| `dSUID` | string (34 hex) | dSUID of vDC or vDC host |
| `params` | PropertyElement[] | - `establish` (boolean): true=learn-in, false=learn-out |

### Authenticate

Initiates authentication process with device.

**Method:** `GenericRequest`

**Direction:** vdSM → vDC host

Uses generic request mechanism with method-specific parameters.

### Firmware Update

Initiates firmware update process for device.

**Method:** `GenericRequest`

**Direction:** vdSM → vDC host

Uses generic request mechanism with update-specific parameters.

### Change Device Configuration/Profile

Changes dS device configuration or profile.

**Method:** `GenericRequest`

**Direction:** vdSM → vDC host

Uses generic request mechanism with configuration parameters.

### Identify vDC Host Device

Makes the vDC host device physically identifiable.

**Notification:** `identify`

**Direction:** vdSM → vDC host

**Target:** vDC host dSUID (not device dSUID)

## Presence Polling

### Ping

Checks if an entity is active and reachable.

**Notification:** `ping`

**Direction:** vdSM → vDC host

**Message:** `vdsm_SendPing`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex) | dSUID of entity to ping (vDC host, vDC, or device) |

#### Available For
- vDC host
- Logical vDCs
- All devices

### Pong

Response to ping notification.

**Notification:** `pong`

**Direction:** vDC host → vdSM

**Message:** `vdc_SendPong`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex) | dSUID of entity responding |

#### Implementation Notes
- Return pong only if entity can be considered active
- If possible, test connection with device hardware
- For unidirectional sensors, use reasonable heuristics
- Part of keep-alive mechanism for session maintenance

## Implementation Examples

### Example 1: Call Scene

```python
def call_scene(session, device_dsuids, scene_number, force=False):
    """Call a scene on one or more devices"""
    notification = vdsm_NotificationCallScene()
    notification.dSUID.extend(device_dsuids)
    notification.scene = scene_number
    notification.force = force
    
    session.send_notification(notification)
```

### Example 2: Dim Channel

```python
def start_dimming_up(session, device_dsuid, channel=0):
    """Start dimming a channel upwards"""
    notification = vdsm_NotificationDimChannel()
    notification.dSUID.append(device_dsuid)
    notification.channel = channel
    notification.mode = 1  # Dim up
    notification.area = 0  # No restriction
    
    session.send_notification(notification)

def stop_dimming(session, device_dsuid, channel=0):
    """Stop dimming"""
    notification = vdsm_NotificationDimChannel()
    notification.dSUID.append(device_dsuid)
    notification.channel = channel
    notification.mode = 0  # Stop
    
    session.send_notification(notification)
```

### Example 3: Set Output Value with Buffering

```python
def set_light_scene(session, devices, values):
    """Set multiple lights with synchronized application"""
    # Buffer all values
    for device, value in zip(devices[:-1], values[:-1]):
        notification = vdsm_NotificationSetOutputChannelValue()
        notification.dSUID.append(device)
        notification.channel = 0
        notification.value = value
        notification.apply_now = False  # Buffer
        session.send_notification(notification)
    
    # Apply all buffered values with the last one
    notification = vdsm_NotificationSetOutputChannelValue()
    notification.dSUID.append(devices[-1])
    notification.channel = 0
    notification.value = values[-1]
    notification.apply_now = True  # Apply all
    session.send_notification(notification)
```

### Example 4: Presence Check

```python
def check_device_presence(session, device_dsuid, timeout=5.0):
    """Check if device is present and active"""
    ping_msg = vdsm_SendPing()
    ping_msg.dSUID = device_dsuid
    
    # Send ping
    session.send_notification(ping_msg)
    
    # Wait for pong
    pong = session.wait_for_pong(device_dsuid, timeout)
    return pong is not None
```

### Example 5: Identify Device

```python
def identify_device(session, device_dsuid):
    """Make device identify itself (e.g., blink LED)"""
    notification = vdsm_NotificationIdentify()
    notification.dSUID.append(device_dsuid)
    
    session.send_notification(notification)
```

## Best Practices

### For vDC Implementers

1. **Scene Support:** Implement all required scene operations for your device type
2. **Dimming:** Handle start/stop dimming appropriately; respect area restrictions
3. **Identify:** Provide clear visual/audio identification
4. **Buffering:** Implement output buffering for synchronized changes
5. **Presence:** Respond to ping with accurate device status

### For vdSM Implementers

1. **Separate Calls:** vdSM creates separate calls for each device in zone/group operations
2. **Error Handling:** Notifications typically don't expect responses; errors may be silent
3. **Timing:** Consider device response times for dimming operations
4. **Presence Polling:** Use ping/pong for keep-alive and device discovery

## Next Steps

- Review [Properties](properties.md) for reading/writing device state
- Explore [Device Operations](device-operations.md) for device management
- See vDC-API-properties specification for action descriptions and parameters
