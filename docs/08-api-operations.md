# API Operations

## Overview

This document describes all API operations in detail: scene operations, channel operations, and device control.

## Scene Operations

### Call Scene

Activates a scene on one or more devices.

**Message**: `vdsm_NotificationCallScene`

```protobuf
message vdsm_NotificationCallScene {
    repeated string dSUID = 1;   // Target devices
    optional int32 scene = 2;    // Scene number (0-127)
    optional bool force = 3;     // Force application
    optional int32 group = 4;    // Functional group
    optional int32 zone_id = 5;  // Zone ID
}
```

**Parameters**:
- `dSUID`: List of device dSUIDs to apply scene to
- `scene`: Scene number to call
- `force`: If true, apply even if already active
- `group`: Optional group filter
- `zone_id`: Optional zone filter

**Implementation**:
```python
def handle_call_scene(self, notification):
    scene_no = notification.scene
    force = notification.force
    
    for dsuid in notification.dSUID:
        device = self.find_device(dsuid)
        if device:
            # Get scene configuration
            scene_config = device.get_scene(scene_no)
            
            if scene_config:
                # Apply channel values from scene
                for channel_id, value in scene_config.items():
                    device.set_channel(channel_id, value)
            else:
                # Scene not configured - use defaults
                if scene_no == 0:  # Off
                    device.set_all_channels(0)
```

**Common Scene Numbers**:
- 0: Off
- 5: Scene 1 / Preset 1
- 17: Scene 2 / Preset 2
- 18: Scene 3 / Preset 3
- 19: Scene 4 / Preset 4

### Save Scene

Saves current device state as a scene.

**Message**: `vdsm_NotificationSaveScene`

```protobuf
message vdsm_NotificationSaveScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Implementation**:
```python
def handle_save_scene(self, notification):
    scene_no = notification.scene
    
    for dsuid in notification.dSUID:
        device = self.find_device(dsuid)
        if device:
            # Capture current channel values
            current_state = {}
            for channel in device.channels:
                current_state[channel.id] = channel.current_value
            
            # Save as scene
            device.save_scene(scene_no, current_state)
            
            # Persist to storage
            self.storage.save_scene(device.dsuid, scene_no, current_state)
```

### Undo Scene

Reverts to state before last scene call.

**Message**: `vdsm_NotificationUndoScene`

```protobuf
message vdsm_NotificationUndoScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Implementation**:
```python
def handle_undo_scene(self, notification):
    for dsuid in notification.dSUID:
        device = self.find_device(dsuid)
        if device and device.has_previous_state():
            # Restore previous state
            previous = device.get_previous_state()
            for channel_id, value in previous.items():
                device.set_channel(channel_id, value)
```

### Set Local Priority

Sets local priority mode for devices.

**Message**: `vdsm_NotificationSetLocalPrio`

```protobuf
message vdsm_NotificationSetLocalPrio {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage**: Advanced feature for priority control between local and remote commands.

### Call Minimum Scene

Calls a scene with minimum value guarantee.

**Message**: `vdsm_NotificationCallMinScene`

```protobuf
message vdsm_NotificationCallMinScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Implementation**:
Similar to CallScene but ensures minimum brightness/value.

## Channel Operations

### Set Output Channel Value

Sets a channel to a specific value.

**Message**: `vdsm_NotificationSetOutputChannelValue`

```protobuf
message vdsm_NotificationSetOutputChannelValue {
    repeated string dSUID = 1;
    optional bool apply_now = 2 [ default = true ];
    optional int32 channel = 3;      // Deprecated
    optional double value = 4;       // Target value
    optional string channelId = 5;   // Channel ID (API v3+)
}
```

**Parameters**:
- `dSUID`: Target devices
- `apply_now`: Apply immediately (true) or stage (false)
- `channelId`: Channel identifier (use this in API v3)
- `value`: Target value (typically 0-100 for percentages)

**Implementation**:
```python
def handle_set_channel_value(self, notification):
    channel_id = notification.channelId
    value = notification.value
    apply_now = notification.apply_now
    
    for dsuid in notification.dSUID:
        device = self.find_device(dsuid)
        if device:
            channel = device.get_channel(channel_id)
            if channel:
                # Clamp value to valid range
                clamped = max(channel.min, min(channel.max, value))
                
                if apply_now:
                    device.set_channel_immediately(channel_id, clamped)
                else:
                    device.stage_channel_value(channel_id, clamped)
```

**Example**: Set brightness to 75%
```json
{
  "dSUID": ["DEVICE1"],
  "channelId": "brightness",
  "value": 75.0,
  "apply_now": true
}
```

### Dim Channel

Starts or stops dimming a channel up or down.

**Message**: `vdsm_NotificationDimChannel`

```protobuf
message vdsm_NotificationDimChannel {
    repeated string dSUID = 1;
    optional int32 channel = 2;      // Deprecated
    optional int32 mode = 3;         // 0=stop, 1=start
    optional int32 area = 4;         // 1=up, 2=down
    optional int32 group = 5;
    optional int32 zone_id = 6;
    optional string channelId = 7;   // API v3+
}
```

**Parameters**:
- `mode`: 0 = stop dimming, 1 = start dimming
- `area`: 1 = dim up, 2 = dim down
- `channelId`: Channel to dim

**Implementation**:
```python
def handle_dim_channel(self, notification):
    channel_id = notification.channelId
    mode = notification.mode
    area = notification.area
    
    for dsuid in notification.dSUID:
        device = self.find_device(dsuid)
        if device:
            if mode == 1:  # Start dimming
                direction = "up" if area == 1 else "down"
                device.start_dimming(channel_id, direction)
            else:  # Stop dimming
                device.stop_dimming(channel_id)

class Device:
    def start_dimming(self, channel_id, direction):
        """Start continuous dimming"""
        self.dimming_active = True
        self.dim_direction = direction
        self.dim_channel = channel_id
        
        # Start dimming thread/task
        self.dim_task = asyncio.create_task(self._dim_loop())
    
    async def _dim_loop(self):
        """Dimming loop"""
        step = 2.0  # 2% per step
        interval = 0.1  # 100ms between steps
        
        while self.dimming_active:
            current = self.get_channel_value(self.dim_channel)
            
            if self.dim_direction == "up":
                new_value = min(100.0, current + step)
            else:
                new_value = max(0.0, current - step)
            
            self.set_channel(self.dim_channel, new_value)
            
            # Check boundaries
            if new_value == 0.0 or new_value == 100.0:
                self.dimming_active = False
            
            await asyncio.sleep(interval)
    
    def stop_dimming(self, channel_id):
        """Stop dimming"""
        self.dimming_active = False
```

## Control Operations

### Set Control Value

Sets a named control value (advanced feature).

**Message**: `vdsm_NotificationSetControlValue`

```protobuf
message vdsm_NotificationSetControlValue {
    repeated string dSUID = 1;
    optional string name = 2;      // Control name
    optional double value = 3;     // Control value
    optional int32 group = 4;
    optional int32 zone_id = 5;
}
```

**Usage**: For advanced control values beyond standard channels.

### Identify

Requests device to identify itself (blink, beep, etc.).

**Message**: `vdsm_NotificationIdentify`

```protobuf
message vdsm_NotificationIdentify {
    repeated string dSUID = 1;
    optional int32 group = 2;
    optional int32 zone_id = 3;
}
```

**Implementation**:
```python
def handle_identify(self, notification):
    for dsuid in notification.dSUID:
        device = self.find_device(dsuid)
        if device:
            device.identify()

class Device:
    def identify(self):
        """Make device identify itself"""
        # Example for light: blink 3 times
        original_brightness = self.get_channel_value("brightness")
        
        for i in range(3):
            self.set_channel("brightness", 100.0)
            time.sleep(0.3)
            self.set_channel("brightness", 0.0)
            time.sleep(0.3)
        
        # Restore original state
        self.set_channel("brightness", original_brightness)
```

## Generic Request (API v2c+)

Generic method invocation mechanism.

**Message**: `vdsm_RequestGenericRequest`

```protobuf
message vdsm_RequestGenericRequest {
    optional string dSUID = 1;
    optional string methodname = 2;
    repeated PropertyElement params = 3;
}
```

**Response**: `GenericResponse`

**Example**: Custom method call
```json
{
  "dSUID": "DEVICE1",
  "methodname": "startEffect",
  "params": [
    { "name": "effect", "value": { "v_string": "rainbow" } },
    { "name": "speed", "value": { "v_uint64": 50 } }
  ]
}
```

**Implementation**:
```python
def handle_generic_request(self, message):
    request = message.vdsm_request_generic_request
    device = self.find_device(request.dSUID)
    
    if not device:
        return self.error_response(ERR_NOT_FOUND)
    
    method_name = request.methodname
    
    # Parse parameters
    params = {}
    for param in request.params:
        params[param.name] = self.extract_value(param.value)
    
    # Call method
    try:
        result = device.call_method(method_name, params)
        return self.success_response()
    except NotImplementedError:
        return self.error_response(ERR_NOT_IMPLEMENTED)
    except Exception as e:
        return self.error_response(ERR_FORBIDDEN, str(e))
```

## Best Practices

### Scene Implementation

1. **Provide defaults**: Have sensible defaults for unconfigured scenes
2. **Persist user scenes**: Save user-configured scenes
3. **Smooth transitions**: Apply scene changes smoothly
4. **Track previous state**: For undo functionality

### Channel Operations

1. **Validate ranges**: Clamp values to min/max
2. **Smooth dimming**: Use appropriate step size and timing
3. **Stop on boundary**: Stop dimming at 0% or 100%
4. **Concurrent operations**: Handle multiple simultaneous requests

### Error Handling

Return appropriate errors:

```python
# Device offline
if not device.is_online():
    return GenericResponse(
        code=ERR_SERVICE_NOT_AVAILABLE,
        description="Device offline"
    )

# Invalid channel
if not device.has_channel(channel_id):
    return GenericResponse(
        code=ERR_NOT_FOUND,
        description=f"Channel '{channel_id}' not found"
    )

# Value out of range
if value < min or value > max:
    return GenericResponse(
        code=ERR_INVALID_VALUE_TYPE,
        description=f"Value must be between {min} and {max}"
    )
```

## Related Documents

- [Protocol Buffer Reference](./09-protobuf-reference.md) - Message definitions
- [Device Features](./07-device-features.md) - Feature descriptions
- [Implementation Guide](./10-implementation-guide.md) - Implementation examples

---

[Back to Documentation Index](./README.md)
