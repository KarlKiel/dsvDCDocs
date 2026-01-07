# Protocol Buffer Messages Reference

## Overview

This document provides a complete reference for all Protocol Buffer messages defined in `genericVDC.proto`. It serves as a detailed specification for implementing the vDC API.

## Message Categories

Messages are organized into these categories:

1. **Session Management**: Connection setup and teardown
2. **Property Access**: Reading and writing device properties
3. **Device Lifecycle**: Announcing and removing devices
4. **Notifications**: Scene and control operations
5. **Keep-Alive**: Connection monitoring
6. **Responses**: Generic and specific responses

## Common Types

### PropertyValue

Represents a typed value in the property tree:

```protobuf
message PropertyValue {
    optional bool v_bool = 1;
    optional uint64 v_uint64 = 2;
    optional int64 v_int64 = 3;
    optional double v_double = 4;
    optional string v_string = 5;
    optional bytes v_bytes = 6;
}
```

**Usage**: Exactly one field should be set based on the property type.

### PropertyElement

Represents a node in the property tree:

```protobuf
message PropertyElement {
    optional string name = 1;              // Property name
    optional PropertyValue value = 2;      // Value (for leaf nodes)
    repeated PropertyElement elements = 3; // Children (for containers)
}
```

**Usage**:
- **Leaf node**: Set `name` and `value`
- **Container**: Set `name` and `elements`
- **Array element**: `name` is the index as a string ("0", "1", etc.)

See [Property System](./04-property-system.md) for detailed usage.

### GenericResponse

Standard response for many operations:

```protobuf
message GenericResponse {
    required ResultCode code = 1 [ default = ERR_OK ];
    optional string description = 2;
}
```

**Fields**:
- `code`: Result status (see ResultCode enum)
- `description`: Optional human-readable error description

## Session Management Messages

### vdsm_RequestHello

**Direction**: vdSM → vDC Host  
**Response**: vdc_ResponseHello

Initiates a session between vdSM and vDC host.

```protobuf
message vdsm_RequestHello {
    optional string dSUID = 1;       // vdSM's unique identifier
    optional uint32 api_version = 2; // Requested API version
}
```

**Fields**:
- `dSUID`: The unique identifier of the vdSM
- `api_version`: API version the vdSM supports (typically 3)

**Example**:
```json
{
  "dSUID": "AAAA00000000000000000000001",
  "api_version": 3
}
```

### vdc_ResponseHello

**Direction**: vDC Host → vdSM  
**Response to**: vdsm_RequestHello

Response to the hello handshake.

```protobuf
message vdc_ResponseHello {
    optional string dSUID = 1;  // vDC host's unique identifier
}
```

**Fields**:
- `dSUID`: The unique identifier of the vDC host

**Example**:
```json
{
  "dSUID": "BBBB00000000000000000000001"
}
```

### vdsm_SendBye

**Direction**: vdSM → vDC Host  
**Response**: None

Requests session termination.

```protobuf
message vdsm_SendBye {
    optional string dSUID = 1;  // vdSM's dSUID
}
```

**Usage**: vDC host should clean up resources and close connection.

## Property Access Messages

### vdsm_RequestGetProperty

**Direction**: vdSM → vDC Host  
**Response**: vdc_ResponseGetProperty

Requests property values from a vDC or device.

```protobuf
message vdsm_RequestGetProperty {
    optional string dSUID = 1;          // Target dSUID (vDC or device)
    repeated PropertyElement query = 2; // Query specification
}
```

**Fields**:
- `dSUID`: Target entity (vDC or device dSUID)
- `query`: Specifies which properties to retrieve

**Query Patterns**: See [Property System](./04-property-system.md#reading-properties)

**Example**:
```json
{
  "dSUID": "DEVICE-001",
  "query": [
    { "name": "name" },
    { "name": "dSUID" }
  ]
}
```

### vdc_ResponseGetProperty

**Direction**: vDC Host → vdSM  
**Response to**: vdsm_RequestGetProperty

Returns requested property values.

```protobuf
message vdc_ResponseGetProperty {
    repeated PropertyElement properties = 1;
}
```

**Fields**:
- `properties`: Property values matching the query structure

**Example**:
```json
{
  "properties": [
    { "name": "name", "value": { "v_string": "Living Room Light" } },
    { "name": "dSUID", "value": { "v_string": "DEVICE-001" } }
  ]
}
```

### vdsm_RequestSetProperty

**Direction**: vdSM → vDC Host  
**Response**: GenericResponse

Sets property values on a vDC or device.

```protobuf
message vdsm_RequestSetProperty {
    optional string dSUID = 1;             // Target dSUID
    repeated PropertyElement properties = 2; // Properties to set
}
```

**Fields**:
- `dSUID`: Target entity
- `properties`: Properties and values to set

**Example**:
```json
{
  "dSUID": "DEVICE-001",
  "properties": [
    { "name": "name", "value": { "v_string": "Bedroom Light" } }
  ]
}
```

### vdc_SendPushProperty

**Direction**: vDC Host → vdSM  
**Response**: Error response only

Pushes property changes to vdSM (older API, use PushNotification instead).

```protobuf
message vdc_SendPushProperty {
    optional string dSUID = 1;
    repeated PropertyElement properties = 2;
}
```

**Note**: Deprecated in favor of vdc_SendPushNotification.

### vdc_SendPushNotification

**Direction**: vDC Host → vdSM  
**Response**: Error response only

Notifies vdSM of property changes and device events.

```protobuf
message vdc_SendPushNotification {
    optional string dSUID = 1;
    repeated PropertyElement changedproperties = 2;
    repeated PropertyElement deviceevents = 3;  // API v2c+
}
```

**Fields**:
- `dSUID`: Source device or vDC dSUID
- `changedproperties`: Properties that have changed
- `deviceevents`: Device events (API v2c and later)

**Usage**: Send when device state changes (button pressed, sensor value updated, etc.)

**Example**:
```json
{
  "dSUID": "DEVICE-001",
  "changedproperties": [
    {
      "name": "inputs",
      "elements": [
        {
          "name": "0",
          "elements": [
            { "name": "state", "value": { "v_uint64": 1 } }
          ]
        }
      ]
    }
  ]
}
```

## Device Lifecycle Messages

### vdc_SendAnnounceVdc

**Direction**: vDC Host → vdSM  
**Response**: Error response only

Announces a new vDC to the system.

```protobuf
message vdc_SendAnnounceVdc {
    optional string dSUID = 1;  // vDC's unique identifier
}
```

**Usage**: Send once per vDC after session establishment. vdSM will then query vDC properties.

### vdc_SendAnnounceDevice

**Direction**: vDC Host → vdSM  
**Response**: Error response only

Announces a new device to the system.

```protobuf
message vdc_SendAnnounceDevice {
    optional string dSUID = 1;      // Device's unique identifier
    optional string vdc_dSUID = 2;  // Parent vDC's dSUID
}
```

**Fields**:
- `dSUID`: Unique device identifier
- `vdc_dSUID`: The vDC that manages this device

**Usage**: Send once per device. vdSM will then query device properties.

### vdc_SendVanish

**Direction**: vDC Host → vdSM  
**Response**: Error response only

Notifies that a device is no longer available.

```protobuf
message vdc_SendVanish {
    optional string dSUID = 1;  // Device's dSUID
}
```

**Usage**: Send when a device goes offline or is removed.

### vdsm_SendRemove

**Direction**: vdSM → vDC Host  
**Response**: None

Requests removal of a device from the system.

```protobuf
message vdsm_SendRemove {
    optional string dSUID = 1;  // Device to remove
}
```

**Usage**: vDC host should clean up device resources.

### vdc_SendIdentify

**Direction**: vDC Host → vdSM  
**Response**: Error response only

Requests visual/audio identification of the vDC host.

```protobuf
message vdc_SendIdentify {
    optional string dSUID = 1;  // vDC host dSUID
}
```

**Usage**: Rare. Used for physical identification of vDC host hardware.

## Keep-Alive Messages

### vdsm_SendPing

**Direction**: vdSM → vDC Host  
**Response**: vdc_SendPong

Checks if connection is alive.

```protobuf
message vdsm_SendPing {
    optional string dSUID = 1;  // Target dSUID
}
```

**Usage**: vdSM sends periodically. vDC host must respond promptly.

### vdc_SendPong

**Direction**: vDC Host → vdSM  
**Response to**: vdsm_SendPing

Response to ping.

```protobuf
message vdc_SendPong {
    optional string dSUID = 1;  // Responder's dSUID
}
```

## Scene Notification Messages

All scene notifications: vdSM → vDC Host, no response expected.

### vdsm_NotificationCallScene

Requests activation of a scene.

```protobuf
message vdsm_NotificationCallScene {
    repeated string dSUID = 1;   // Target device(s)
    optional int32 scene = 2;    // Scene number
    optional bool force = 3;     // Force scene call
    optional int32 group = 4;    // Device group
    optional int32 zone_id = 5;  // Zone ID
}
```

**Fields**:
- `dSUID`: List of device dSUIDs to apply scene to
- `scene`: Scene number (0-127)
- `force`: Force scene application even if already active
- `group`: Functional group (color)
- `zone_id`: Zone identifier

**Usage**: Device should apply the specified scene configuration.

**Common Scene Numbers**:
- 0: Off
- 5: Scene 1 (preset 1)
- 17: Scene 2 (preset 2)
- etc.

### vdsm_NotificationSaveScene

Requests saving current state as a scene.

```protobuf
message vdsm_NotificationSaveScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage**: Device should save its current state as the specified scene.

### vdsm_NotificationUndoScene

Requests reverting to previous state before scene was called.

```protobuf
message vdsm_NotificationUndoScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage**: Device should restore previous state.

### vdsm_NotificationSetLocalPrio

Sets local priority for a scene.

```protobuf
message vdsm_NotificationSetLocalPrio {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage**: Sets local priority mode for devices.

### vdsm_NotificationCallMinScene

Calls a scene with minimum value guarantee.

```protobuf
message vdsm_NotificationCallMinScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage**: Like CallScene but ensures minimum brightness/value.

## Control Notification Messages

### vdsm_NotificationIdentify

Requests device identification (blink, beep, etc.).

```protobuf
message vdsm_NotificationIdentify {
    repeated string dSUID = 1;
    optional int32 group = 2;
    optional int32 zone_id = 3;
}
```

**Usage**: Device should identify itself (flash light, beep, etc.).

### vdsm_NotificationSetControlValue

Sets a named control value.

```protobuf
message vdsm_NotificationSetControlValue {
    repeated string dSUID = 1;
    optional string name = 2;     // Control name
    optional double value = 3;    // Control value
    optional int32 group = 4;
    optional int32 zone_id = 5;
}
```

**Usage**: Sets custom control values (advanced feature).

### vdsm_NotificationDimChannel

Dims a channel up or down.

```protobuf
message vdsm_NotificationDimChannel {
    repeated string dSUID = 1;
    optional int32 channel = 2;   // Channel index (deprecated)
    optional int32 mode = 3;      // Dim mode (start/stop)
    optional int32 area = 4;      // Dim direction (up/down)
    optional int32 group = 5;
    optional int32 zone_id = 6;
    optional string channelId = 7; // Channel ID (API v3+)
}
```

**Fields**:
- `channel`: Channel index (use channelId instead in v3)
- `mode`: 0 = stop dimming, 1 = start dimming
- `area`: Dim direction (1 = up, 2 = down)
- `channelId`: Channel identifier string (API v3)

**Usage**: Start/stop dimming animation on a channel.

### vdsm_NotificationSetOutputChannelValue

Sets an output channel to a specific value.

```protobuf
message vdsm_NotificationSetOutputChannelValue {
    repeated string dSUID = 1;
    optional bool apply_now = 2 [ default = true ];
    optional int32 channel = 3;    // Channel index (deprecated)
    optional double value = 4;     // Channel value
    optional string channelId = 5; // Channel ID (API v3+)
}
```

**Fields**:
- `apply_now`: Apply immediately (true) or stage for later (false)
- `channel`: Channel index (use channelId in v3)
- `value`: Target value (typically 0.0-100.0)
- `channelId`: Channel identifier (API v3)

**Usage**: Set channel to specific value (e.g., brightness to 75%).

## Generic Request (API v2c+)

### vdsm_RequestGenericRequest

**Direction**: vdSM → vDC Host  
**Response**: GenericResponse

Generic method call mechanism.

```protobuf
message vdsm_RequestGenericRequest {
    optional string dSUID = 1;
    optional string methodname = 2;
    repeated PropertyElement params = 3;
}
```

**Fields**:
- `dSUID`: Target entity
- `methodname`: Method to call
- `params`: Method parameters

**Usage**: Extensibility mechanism for custom methods.

## Enumerations

### Type Enum

All message types:

```protobuf
enum Type {
    GENERIC_RESPONSE = 1;
    
    VDSM_REQUEST_HELLO = 2;
    VDC_RESPONSE_HELLO = 3;
    
    VDSM_REQUEST_GET_PROPERTY = 4;
    VDC_RESPONSE_GET_PROPERTY = 5;
    
    VDSM_REQUEST_SET_PROPERTY = 6;
    VDC_RESPONSE_SET_PROPERTY = 7;
    
    VDSM_SEND_PING = 8;
    VDC_SEND_PONG = 9;
    VDC_SEND_ANNOUNCE_DEVICE = 10;
    VDC_SEND_VANISH = 11;
    VDC_SEND_PUSH_PROPERTY = 12;
    VDSM_SEND_REMOVE = 13;
    VDSM_SEND_BYE = 14;
    
    VDSM_NOTIFICATION_CALL_SCENE = 15;
    VDSM_NOTIFICATION_SAVE_SCENE = 16;
    VDSM_NOTIFICATION_UNDO_SCENE = 17;
    VDSM_NOTIFICATION_SET_LOCAL_PRIO = 18;
    VDSM_NOTIFICATION_CALL_MIN_SCENE = 19;
    VDSM_NOTIFICATION_IDENTIFY = 20;
    VDSM_NOTIFICATION_SET_CONTROL_VALUE = 21;
    
    VDC_SEND_IDENTIFY = 22;
    VDC_SEND_ANNOUNCE_VDC = 23;
    VDSM_NOTIFICATION_DIM_CHANNEL = 24;
    VDSM_NOTIFICATION_SET_OUTPUT_CHANNEL_VALUE = 25;
    VDSM_REQUEST_GENERIC_REQUEST = 26;
}
```

### ResultCode Enum

Error codes for GenericResponse:

```protobuf
enum ResultCode {
    ERR_OK = 0;                      // Success
    ERR_MESSAGE_UNKNOWN = 1;         // Unknown message type
    ERR_INCOMPATIBLE_API = 2;        // API version not supported
    ERR_SERVICE_NOT_AVAILABLE = 3;   // Service unavailable
    ERR_INSUFFICIENT_STORAGE = 4;    // Out of storage
    ERR_FORBIDDEN = 5;               // Operation not allowed
    ERR_NOT_IMPLEMENTED = 6;         // Not implemented
    ERR_NO_CONTENT_FOR_ARRAY = 7;    // Array access error
    ERR_INVALID_VALUE_TYPE = 8;      // Type mismatch
    ERR_MISSING_SUBMESSAGE = 9;      // Required message missing
    ERR_MISSING_DATA = 10;           // Required data missing
    ERR_NOT_FOUND = 11;              // Resource not found
    ERR_NOT_AUTHORIZED = 12;         // Not authorized
}
```

## Message Wrapper

All messages are wrapped in the Message type:

```protobuf
message Message {
    required Type type = 1 [ default = GENERIC_RESPONSE ];
    optional uint32 message_id = 2 [ default = 0 ];
    optional GenericResponse generic_response = 3;
    
    // Request/Response pairs
    optional vdsm_RequestHello vdsm_request_hello = 100;
    optional vdc_ResponseHello vdc_response_hello = 101;
    optional vdsm_RequestGetProperty vdsm_request_get_property = 102;
    optional vdc_ResponseGetProperty vdc_response_get_property = 103;
    optional vdsm_RequestSetProperty vdsm_request_set_property = 104;
    optional vdsm_RequestGenericRequest vdsm_request_generic_request = 123;
    
    // Send messages
    optional vdsm_SendPing vdsm_send_ping = 105;
    optional vdc_SendPong vdc_send_pong = 106;
    optional vdc_SendAnnounceDevice vdc_send_announce_device = 107;
    optional vdc_SendVanish vdc_send_vanish = 108;
    optional vdc_SendPushProperty vdc_send_push_property = 109;
    optional vdsm_SendRemove vdsm_send_remove = 110;
    optional vdsm_SendBye vdsm_send_bye = 111;
    
    // Notifications
    optional vdsm_NotificationCallScene vdsm_send_call_scene = 112;
    optional vdsm_NotificationSaveScene vdsm_send_save_scene = 113;
    optional vdsm_NotificationUndoScene vdsm_send_undo_scene = 114;
    optional vdsm_NotificationSetLocalPrio vdsm_send_set_local_prio = 115;
    optional vdsm_NotificationCallMinScene vdsm_send_call_min_scene = 116;
    optional vdsm_NotificationIdentify vdsm_send_identify = 117;
    optional vdsm_NotificationSetControlValue vdsm_send_set_control_value = 118;
    optional vdsm_NotificationDimChannel vdsm_send_dim_channel = 121;
    optional vdsm_NotificationSetOutputChannelValue vdsm_send_output_channel_value = 122;
    
    // Additional
    optional vdc_SendIdentify vdc_send_identify = 119;
    optional vdc_SendAnnounceVdc vdc_send_announce_vdc = 120;
}
```

## Quick Reference Tables

### Message Direction Quick Reference

| From vdSM | From vDC Host |
|-----------|---------------|
| RequestHello | ResponseHello |
| RequestGetProperty | ResponseGetProperty |
| RequestSetProperty | GenericResponse |
| RequestGenericRequest | GenericResponse |
| SendPing | SendPong |
| SendBye | - |
| SendRemove | - |
| Notification* (all) | - |
| - | SendAnnounceVdc |
| - | SendAnnounceDevice |
| - | SendVanish |
| - | SendPushNotification |
| - | SendIdentify |

### Response Type Quick Reference

| Request/Send | Response |
|--------------|----------|
| vdsm_RequestHello | vdc_ResponseHello |
| vdsm_RequestGetProperty | vdc_ResponseGetProperty |
| vdsm_RequestSetProperty | GenericResponse |
| vdsm_RequestGenericRequest | GenericResponse |
| vdsm_SendPing | vdc_SendPong |
| vdc_Send* (announce, vanish, etc.) | Error response only |
| vdsm_Notification* | No response |
| vdsm_SendBye | No response |
| vdsm_SendRemove | No response |

## Implementation Notes

### Required Messages

Minimum implementation must support:
- Hello handshake
- AnnounceVdc, AnnounceDevice
- GetProperty, SetProperty
- Ping/Pong
- At least one notification type (typically CallScene)

### Optional Messages

Can be implemented as needed:
- All scene operations
- Dimming operations
- Generic request
- Control values
- Identify

### Message ID Usage

- Set `message_id` for all requests
- Copy `message_id` to corresponding response
- Use 0 for messages without response
- Increment for each new request

### Error Handling

Always return appropriate error responses:
- Check message validity
- Validate dSUID
- Check property existence
- Return specific error codes

## See Also

- [Property System](./04-property-system.md) - Understanding PropertyElement usage
- [Protocol Basics](./03-protocol-basics.md) - Connection and message flow
- [API Operations](./08-api-operations.md) - Detailed operation descriptions

---

[Back to Documentation Index](./README.md)
