# Protocol Buffer Reference

## Overview

This document provides a complete reference for the vDC-API Protocol Buffer messages defined in `genericVDC.proto`.

## Message Type Enum

All messages in the vDC-API are wrapped in a `Message` container with a type identifier:

```protobuf
enum Type {
    GENERIC_RESPONSE = 1;
    
    // Session Management
    VDSM_REQUEST_HELLO = 2;
    VDC_RESPONSE_HELLO = 3;
    
    // Property Access
    VDSM_REQUEST_GET_PROPERTY = 4;
    VDC_RESPONSE_GET_PROPERTY = 5;
    VDSM_REQUEST_SET_PROPERTY = 6;
    VDC_RESPONSE_SET_PROPERTY = 7;
    
    // Presence Polling
    VDSM_SEND_PING = 8;
    VDC_SEND_PONG = 9;
    
    // Device Management
    VDC_SEND_ANNOUNCE_DEVICE = 10;
    VDC_SEND_VANISH = 11;
    VDC_SEND_PUSH_PROPERTY = 12;
    VDSM_SEND_REMOVE = 13;
    VDSM_SEND_BYE = 14;
    
    // Scene Operations
    VDSM_NOTIFICATION_CALL_SCENE = 15;
    VDSM_NOTIFICATION_SAVE_SCENE = 16;
    VDSM_NOTIFICATION_UNDO_SCENE = 17;
    VDSM_NOTIFICATION_SET_LOCAL_PRIO = 18;
    VDSM_NOTIFICATION_CALL_MIN_SCENE = 19;
    
    // Device Actions
    VDSM_NOTIFICATION_IDENTIFY = 20;
    VDSM_NOTIFICATION_SET_CONTROL_VALUE = 21;
    VDC_SEND_IDENTIFY = 22;
    VDC_SEND_ANNOUNCE_VDC = 23;
    VDSM_NOTIFICATION_DIM_CHANNEL = 24;  // API v2
    VDSM_NOTIFICATION_SET_OUTPUT_CHANNEL_VALUE = 25;
    VDSM_REQUEST_GENERIC_REQUEST = 26;  // API v2c
}
```

## Result Codes

```protobuf
enum ResultCode {
    ERR_OK = 0;
    ERR_MESSAGE_UNKNOWN = 1;
    ERR_INCOMPATIBLE_API = 2;
    ERR_SERVICE_NOT_AVAILABLE = 3;
    ERR_INSUFFICIENT_STORAGE = 4;
    ERR_FORBIDDEN = 5;
    ERR_NOT_IMPLEMENTED = 6;
    ERR_NO_CONTENT_FOR_ARRAY = 7;
    ERR_INVALID_VALUE_TYPE = 8;
    ERR_MISSING_SUBMESSAGE = 9;
    ERR_MISSING_DATA = 10;
    ERR_NOT_FOUND = 11;
    ERR_NOT_AUTHORIZED = 12;
}
```

## Core Data Structures

### PropertyValue

Represents a single value of various types:

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

### PropertyElement

Hierarchical property structure:

```protobuf
message PropertyElement {
    optional string name = 1;
    optional PropertyValue value = 2;
    repeated PropertyElement elements = 3;
}
```

### GenericResponse

Standard error/success response:

```protobuf
message GenericResponse {
    required ResultCode code = 1 [default = ERR_OK];
    optional string description = 2;
}
```

## Session Management Messages

### vdsm_RequestHello

Initiates vDC session:

```protobuf
message vdsm_RequestHello {
    optional string dSUID = 1;        // vdSM dSUID
    optional uint32 api_version = 2;  // API version
}
```

**Usage:** vdSM → vDC host
**Response:** `vdc_ResponseHello` or `GenericResponse` (error)

### vdc_ResponseHello

Session initialization response:

```protobuf
message vdc_ResponseHello {
    optional string dSUID = 1;  // vDC host dSUID
}
```

**Usage:** vDC host → vdSM

### vdsm_SendBye

Terminates vDC session:

```protobuf
message vdsm_SendBye {
    optional string dSUID = 1;  // vDC host dSUID
}
```

**Usage:** vdSM → vDC host
**Response:** `GenericResponse`

## Device Announcement Messages

### vdc_SendAnnounceVdc

Announces a logical vDC:

```protobuf
message vdc_SendAnnounceVdc {
    optional string dSUID = 1;  // vDC dSUID
}
```

**Usage:** vDC host → vdSM
**Response:** `GenericResponse`

### vdc_SendAnnounceDevice

Announces a device:

```protobuf
message vdc_SendAnnounceDevice {
    optional string dSUID = 1;      // Device dSUID
    optional string vdc_dSUID = 2;  // Parent vDC dSUID
}
```

**Usage:** vDC host → vdSM
**Response:** `GenericResponse`

### vdc_SendVanish

Notifies device removal:

```protobuf
message vdc_SendVanish {
    optional string dSUID = 1;  // Device dSUID
}
```

**Usage:** vDC host → vdSM
**Response:** `GenericResponse`

### vdsm_SendRemove

Requests device removal:

```protobuf
message vdsm_SendRemove {
    optional string dSUID = 1;  // Device dSUID
}
```

**Usage:** vdSM → vDC host
**Response:** `GenericResponse`

## Property Access Messages

### vdsm_RequestGetProperty

Reads property values:

```protobuf
message vdsm_RequestGetProperty {
    optional string dSUID = 1;
    repeated PropertyElement query = 2;
}
```

**Usage:** vdSM → vDC host
**Response:** `vdc_ResponseGetProperty` or `GenericResponse` (error)

### vdc_ResponseGetProperty

Property read response:

```protobuf
message vdc_ResponseGetProperty {
    repeated PropertyElement properties = 1;
}
```

**Usage:** vDC host → vdSM

### vdsm_RequestSetProperty

Writes property values:

```protobuf
message vdsm_RequestSetProperty {
    optional string dSUID = 1;
    repeated PropertyElement properties = 2;
}
```

**Usage:** vdSM → vDC host
**Response:** `GenericResponse`

### vdc_SendPushNotification

Pushes property changes (API v2c+):

```protobuf
message vdc_SendPushNotification {
    optional string dSUID = 1;
    repeated PropertyElement changedproperties = 2;
    repeated PropertyElement deviceevents = 3;  // API v2c
}
```

**Usage:** vDC host → vdSM

### vdc_SendPushProperty

Legacy property push (pre-v2c):

```protobuf
message vdc_SendPushProperty {
    optional string dSUID = 1;
    repeated PropertyElement properties = 2;
}
```

**Usage:** vDC host → vdSM

## Presence Polling Messages

### vdsm_SendPing

Checks entity presence:

```protobuf
message vdsm_SendPing {
    optional string dSUID = 1;  // Entity dSUID
}
```

**Usage:** vdSM → vDC host

### vdc_SendPong

Presence confirmation:

```protobuf
message vdc_SendPong {
    optional string dSUID = 1;  // Entity dSUID
}
```

**Usage:** vDC host → vdSM

## Scene Operation Messages

### vdsm_NotificationCallScene

Calls a scene:

```protobuf
message vdsm_NotificationCallScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional bool force = 3;
    optional int32 group = 4;
    optional int32 zone_id = 5;
}
```

**Usage:** vdSM → vDC host (no response expected)

### vdsm_NotificationSaveScene

Saves device state to scene:

```protobuf
message vdsm_NotificationSaveScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage:** vdSM → vDC host (no response expected)

### vdsm_NotificationUndoScene

Undoes a scene call:

```protobuf
message vdsm_NotificationUndoScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage:** vdSM → vDC host (no response expected)

### vdsm_NotificationSetLocalPrio

Sets local priority:

```protobuf
message vdsm_NotificationSetLocalPrio {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage:** vdSM → vDC host (no response expected)

### vdsm_NotificationCallMinScene

Calls minimum scene:

```protobuf
message vdsm_NotificationCallMinScene {
    repeated string dSUID = 1;
    optional int32 scene = 2;
    optional int32 group = 3;
    optional int32 zone_id = 4;
}
```

**Usage:** vdSM → vDC host (no response expected)

## Device Action Messages

### vdsm_NotificationIdentify

Identifies device(s):

```protobuf
message vdsm_NotificationIdentify {
    repeated string dSUID = 1;
    optional int32 group = 2;
    optional int32 zone_id = 3;
}
```

**Usage:** vdSM → vDC host (no response expected)

### vdc_SendIdentify

Requests device identification:

```protobuf
message vdc_SendIdentify {
    optional string dSUID = 1;
}
```

**Usage:** vDC host → vdSM or vdSM → vDC host

### vdsm_NotificationSetControlValue

Sets a control value:

```protobuf
message vdsm_NotificationSetControlValue {
    repeated string dSUID = 1;
    optional string name = 2;
    optional double value = 3;
    optional int32 group = 4;
    optional int32 zone_id = 5;
}
```

**Usage:** vdSM → vDC host (no response expected)

### vdsm_NotificationDimChannel

Controls channel dimming (API v2):

```protobuf
message vdsm_NotificationDimChannel {
    repeated string dSUID = 1;
    optional int32 channel = 2;
    optional int32 mode = 3;
    optional int32 area = 4;
    optional int32 group = 5;
    optional int32 zone_id = 6;
    optional string channelId = 7;  // API v3
}
```

**Usage:** vdSM → vDC host (no response expected)

**Mode values:**
- `1` - Start dimming up
- `-1` - Start dimming down
- `0` - Stop dimming

### vdsm_NotificationSetOutputChannelValue

Sets output channel value:

```protobuf
message vdsm_NotificationSetOutputChannelValue {
    repeated string dSUID = 1;
    optional bool apply_now = 2 [default = true];
    optional int32 channel = 3;
    optional double value = 4;
    optional string channelId = 5;  // API v3
}
```

**Usage:** vdSM → vDC host (no response expected)

### vdsm_RequestGenericRequest

Generic method invocation (API v2c):

```protobuf
message vdsm_RequestGenericRequest {
    optional string dSUID = 1;
    optional string methodname = 2;
    repeated PropertyElement params = 3;
}
```

**Usage:** vdSM → vDC host
**Response:** `GenericResponse`

## Message Wrapper

All messages are wrapped in the `Message` container:

```protobuf
message Message {
    required Type type = 1 [default = GENERIC_RESPONSE];
    optional uint32 message_id = 2 [default = 0];
    optional GenericResponse generic_response = 3;
    
    // Request-response messages
    optional vdsm_RequestHello vdsm_request_hello = 100;
    optional vdc_ResponseHello vdc_response_hello = 101;
    optional vdsm_RequestGetProperty vdsm_request_get_property = 102;
    optional vdc_ResponseGetProperty vdc_response_get_property = 103;
    optional vdsm_RequestSetProperty vdsm_request_set_property = 104;
    optional vdsm_RequestGenericRequest vdsm_request_generic_request = 123;
    
    // Send messages (error response only)
    optional vdsm_SendPing vdsm_send_ping = 105;
    optional vdc_SendPong vdc_send_pong = 106;
    optional vdc_SendAnnounceDevice vdc_send_announce_device = 107;
    optional vdc_SendVanish vdc_send_vanish = 108;
    optional vdc_SendPushProperty vdc_send_push_property = 109;
    optional vdsm_SendRemove vdsm_send_remove = 110;
    optional vdsm_SendBye vdsm_send_bye = 111;
    
    // Notifications (no response)
    optional vdsm_NotificationCallScene vdsm_send_call_scene = 112;
    optional vdsm_NotificationSaveScene vdsm_send_save_scene = 113;
    optional vdsm_NotificationUndoScene vdsm_send_undo_scene = 114;
    optional vdsm_NotificationSetLocalPrio vdsm_send_set_local_prio = 115;
    optional vdsm_NotificationCallMinScene vdsm_send_call_min_scene = 116;
    optional vdsm_NotificationIdentify vdsm_send_identify = 117;
    optional vdsm_NotificationSetControlValue vdsm_send_set_control_value = 118;
    optional vdc_SendIdentify vdc_send_identify = 119;
    optional vdc_SendAnnounceVdc vdc_send_announce_vdc = 120;
    optional vdsm_NotificationDimChannel vdsm_send_dim_channel = 121;
    optional vdsm_NotificationSetOutputChannelValue vdsm_send_output_channel_value = 122;
}
```

## Message ID Correlation

The `message_id` field in the `Message` wrapper is used to correlate requests with responses:

- **Sender assigns:** The party sending a request assigns a `message_id`
- **Receiver echoes:** The party responding includes the same `message_id` in the response
- **Optional:** Not required for notifications that don't expect responses
- **Scope:** Unique within the sender's context

## Usage Patterns

### Request-Response Pattern

```
Client                          Server
   |                              |
   |-- Request (message_id=42) -->|
   |                              |
   |<- Response (message_id=42)---|
```

### Notification Pattern

```
Sender                        Receiver
   |                              |
   |-- Notification (no id) ----->|
   |  (no response expected)      |
```

### Error Response Pattern

```
Client                          Server
   |                              |
   |-- Request (message_id=42) -->|
   |                              |
   |<- GenericResponse ------------|
   |   (message_id=42, code≠0)    |
```

## API Version Evolution

### API v2 (Base)
- Core session management
- Property access
- Basic scene operations
- Device announcement

### API v2c
- Added `vdsm_RequestGenericRequest`
- Added `deviceevents` to `vdc_SendPushNotification`

### API v3
- Added `channelId` string field to channel operations
- Allows channel identification by name instead of number

## Next Steps

- Review [API Basics](api-basics.md) for protocol concepts
- Explore [Session Management](session-management.md) for message flow
- See [Properties](properties.md) for property tree usage
- Check [Actions and Notifications](actions-notifications.md) for action messages
