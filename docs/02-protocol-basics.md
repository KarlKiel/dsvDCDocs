
# API Basics

## Introduction

The digitalSTROM Virtual Device Connector API (vDC-API) is a protocol for integrating external devices into the digitalSTROM ecosystem. This document covers the fundamental concepts and conventions used throughout the API.

## Client-Server Architecture

- **Server:** vDC host (your device or gateway)
- **Client:** vdSM (virtual digitalSTROM Meter running on the digitalSTROM Server)
- **Connection:** TCP socket initiated by the vdSM to the vDC host

## Session Lifetime

- A **vDC session** exists for the lifetime of the TCP connection
- If the connection breaks, a new session must be established
- All session state is lost when the connection is terminated
  
## Transport Layer

- **Protocol**: TCP
- **Default Port**: 8440 (configurable via service announcement)
- **Connection**: Initiated by vdSM, accepted by vDC host
- **Encoding**: Protobuf binary messages
- **Framing**: Implementation-specific (often length-prefixed)


### Message Structure
The vdc-API uses **Protocol Buffers (protobuf)** to encode/decode the payload of the communication messages.
  
Protocol Buffers is a language-neutral, platform-neutral mechanism for serializing structured data:

- **Binary encoding**: Efficient on-the-wire representation
- **Schema-based**: Messages defined in `.proto` files (See a .proto representation for this vdC-API in `genericVDC.proto`)
- **Strongly typed**: Type safety with automatic validation
- **Version-tolerant**: New fields can be added without breaking compatibility 
    
All vdC-API communications use messages with a simple framing protocol:

1. **Message Header:** 2 bytes containing the message length
   - Format: 16-bit integer in network byte order (big-endian)
   - Maximum message length: 16,384 bytes (16 KB)

2. **Message Payload:** (as serialized Protocol Buffer message)
   - Contains message type and data:
     General structure for the payload is for all messages to use a Message wrapper:

   ```protobuf
    message Message {                                            // Wrapper (only this Message Type is used as payload)
        required Type type = 1 [ default = GENERIC_RESPONSE ];   // Message Types defined by API (enum within .proto file)
        optional uint32 message_id = 2 [ default = 0 ];          // id for identification and correlation of messages
        optional <MessageType> <messageType> = xx;               // defined message of type <MessageType> as described in this API (and represented in .prot file)
        }
   ```
   
     MessageType names are prefixed to indicate direction:
     - **vdsm_**: Message sent by vdSM (dSS) to vDC host
     - **vdc_**: Message sent by vDC host to vdSM (dSS)

### Message Categories

Messages fall into several categories:

|          Category           |        Description           |
|-----------------------------|------------------------------|
| **Request/Response**        | Expects specific response    |
| **Request/GenericResponse** | Expects GenericResponse      | 
| **Send**                    | May send error response only | 
| **Notification**            | No response expected         | 



## Connection Lifecycle

### 1. Discovery Phase

```
vDC Host                                vdSM
    |                                      |
    | Publish Avahi/mDNS service           |
    | Type: _ds-vdc._tcp                   |
    | Port: 8440                           |
    |                                      |
    |           <-- Discovers service --   |
    |                                      |
```
Service announcement should include:
- Service type: `_ds-vdc._tcp`
- Port number (default 8440)
- Optional TXT records with metadata

### 2. Connection Establishment

```
vDC Host                                vdSM
    |                                      |
    |    <-- TCP Connect (port 8440) --    |
    |                                      |
    | TCP connection established           |
    |                                      |
```

The vdSM initiates the TCP connection to the vDC host.

### 3. Session Initialization (Handshake)

```
vDC Host                                vdSM
    |                                      |
    |    <-- vdsm_RequestHello --------    |
    |        - dSUID: vdSM ID              |
    |        - api_version: 3              |
    |                                      |
    | -- vdc_ResponseHello ----------->    |
    |    - dSUID: vDC host ID              |
    |                                      |
```
The vdSM sends a RequestHello Message to vdC host and expects it to return a ResponseHello with a dSUID as identification

### 4. vDC Announcement

After handshake, announce all vDCs individually:

```
vDC Host                                vdSM
    |                                      |
    | -- vdc_SendAnnounceVdc ---------->   |
    |    - dSUID: vDC #1 dSUID             |
    |                                      |
    | <------- GenericResponse ---------   |
    |                                      |
```
vdC host announces every hosted vDC via individual SendAnnounceVdc messages with vDC dsUID as identifier to vdSM

### 5. Device Announcement

Announce all devices for each vDC:

```
vDC Host                                vdSM
    |                                      |
    | -- vdc_SendAnnounceDevice ------->   |
    |    - dSUID: device dSUID             |
    |    - vdc_dSUID: parent vDC dSUID     |
    |                                      |
    | <------- GenericResponse ---------   |
    |                                      |
```
For every hosted vDC, vDC host will send a SendAnnounceDevice message for every managed device of the respective vDC

### 6. Operational Phase

During the Operational Phase the vdSM and vDC host exchange regularly operational messages: property reads/writes, pushes, pings, actions/notifications (scenes, dimming, identify, etc.). 

1) Get Property 
```
vDC Host                                 vdSM
    |                                      |
    | <-------- vdsm_RequestGetProperty -- |
    |   - dSUID: target entity dSUID       |
    |   - query: (property tree structure) |
    |                                      |
    | --- vdc_ResponseGetProperty ------>  |
    |   - properties: (prop. tree struct.) |
    |                                      |
```
vdSM requests information on vdC or vdSD properties, vdC host sends requested property information

2) Set Property

```
vDC Host                                vdSM
    |                                      |
    | <------- vdsm_RequestSetProperty --- |
    |   - dSUID: target entity dSUID       |
    |   - properties: (prop. tree struct.) |
    |                                      |
    | ------- GenericResponse -----------> |
    |                                      |
```

vdSM requests vdC to set/update (write-enabled) properties

3) Push Property (vDC → vdSM notification)

```
old:                                                 new:
vDC Host                                vdSM         vDC Host                                     vdSM
    |                                      |           |                                            |
    | -- vdc_SendPushProperty ---------->  |           | ------ vdc_SendPushNotification -------->  |
    |   - dSUID: origin entity dSUID       |           |   - dSUID: origin entity dSUID             |
    |   - properties: (prop. tree struct.) |           |   - changedproperties: (prop. tree struct.)|
    |                                      |           |   - deviceevents: ???                      |
    |        (no response expected;        |           |                                            |
    |    error via GenericResponse only)   |           |                    ???                     |
```
vdC pushes information about changes in vdC or vdSD properties to vdsm

4) Generic Requests (Calling specific actions or methods)

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_RequestGenericRequest --- |
    |   - dSUID: target (optional)         |
    |   - method: (enum)                   |
    |   - (method specific payload)        |
    |                                      |
    | ------- GenericResponse -----------> |
    |                                      |
```

vdSM calls vdSD(s) or vdC(s) to call actions or execute specific methods (invokeDeviceAction, pair, authenticate, firmwareUpgrade, setConfiguration, identify)

5) Call Scene Notification

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_NotificationCallScene --- |
    |   - dSUID: one or many device dSUIDs |
    |   - scene: scene number              |
    |   - force: boolean                   |
    |   - group / zoneID: optional         |
    |                                      |
    |     (no response expected)           |
```
vdSM notifies vdC host that a scene call has been made to the dss, that might affect managed vdSD(s)

6) Save Scene Notification

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_NotificationSaveScene --- |
    |   - dSUID: one or many device dSUIDs |
    |   - scene: scene number              |
    |   - group / zoneID: optional         |
    |                                      |
    |       (no response expected)         |
```
vdSM notifies vdC host that a scene configuration has been created, updated or deleted, which might affect managed vdDS(s)

7) Undo Scene Notification

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_NotificationUndoScene --- |
    |   - dSUID: one or many device dSUIDs |
    |   - scene: scene number to undo      |
    |   - group / zoneID: optional         |
    |                                      |
    |      (no response expected)          |
```
vdSM notifies vdC host that there has been an undo-Scene call in the dss, which might affect managed vdSDs

8) Notification of Local Priority Change Request 

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationSetLocalPrio ---|
    |   - dSUID: target dSUID(s)           |
    |   - scene: scene to check            |
    |   - group / zoneID: optional         |
    |                                      |
    |       (no response expected)         |
```
vdSM notifies vdC host about LocalPriority configuration changes for a scene, which might affect managed vdSD(s)

9) Notification of Dimming Requests 

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationDimChannel -----|
    |   - dSUID: device dSUID(s)           |
    |   - channel: channel index           |
    |   - mode: 1 / -1 / 0                 |
    |   - area: area restriction           |
    |                                      |
    |      (no response expected)          |
```
vdSM notifies vdC host about a requested dimming operation (start upwards, start downwards, stop) for a defined channel, which might affect managed vdSD(s)

11) Notification of Call Minimum Scene Request

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationCallMinScene ----|
    |   - dSUID: device dSUID(s)           |
    |   - scene: min scene number          |
    |   - group / zoneID: optional         |
    |                                      |
    |      (no response expected)          |
```
vdSM notifies vdC host about a request to call minimum scen, which meight affect managed vdSD(s)

12) Identification Request

```
vDC Host                                vdSM
    |                                      |
    | <--- vdsm_NotificationIdentify ----- |
    |   - dSUID: device dSUID(s)           |
    |   - parameters: (optional)           |
    |                                      |
    |      (no response expected)          |
```
vdSM notifies vdC host about a request to a device to run their routine for helping human beings to identify a specific device (by blinking , ringing, playing a sound...), which might affect managed vdSD(s)

```
vDC Host                                 vdSM
    |                                      |
    | --------- vdc_SendIdentify --------> |
    |   - dSUID: device dSUID(s)           |
    |                                      |
    |      (no response expected)          |
```
vdc sends vdSM a request to device(s) that might not be managed on vdC host to help human beings to identify them (by blinking , ringing, playing a sound...), which might affect managed vdSD(s)


13) Notification of a set/change of a Control Value

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationSetControlValue -|
    |   - dSUID: device dSUID(s)           |
    |   - control: control identifier      |
    |   - value: value to set              |
    |                                      |
    |      (no response expected)          |
```
vdSM notifies vdC host that a control value has been set/changed, which might affect managed vdSD(s)

14) Notification about a request to set/update an Output Channel Value 

```
vDC Host                                       vdSM
    |                                            |
    | <- vdsm_NotificationSetOutputChannelValue -|
    |   - dSUID: device dSUID(s)                 | 
    |   - channel: output channel                |
    |   - value: target output value             |
    |                                            |
    |         (no response expected              |
```
vdSM notifies vdc host about a request to set the value of an output channel for one or several devices, which might contain managed vdSD(s) 

### 7. Keep-Alive

Periodic ping/pong to verify connection:

```
vDC Host                                vdSM
    |                                      |
    |    <-- vdsm_SendPing -------------   |
    |        - dSUID: target dSUID         |
    |                                      |
    | -- vdc_SendPong ----------------->   |
    |    - dSUID: responder dSUID          |
    |                                      |
```
For connection verification vdSM will periodically send SendPing messages and expects immediate SendPong messages from vdC host in return

### 9. Session Termination

1) Clean shutdown of dss/vdSM:

```
vDC Host                                vdSM
    |                                      |
    |    <-- vdsm_SendBye --------------   |
    |        - dSUID: vdSM dSUID           |
    |                                      |
    | Clean up resources                   |
    | Close TCP connection                 |
    |                                      |
```
To make a clean temporary shutdown, the vdSM sends a SenBye message to the vdC host before shutting down the TCP connection.

2) Device removal (vDC triggered):

```
vDC Host                                vdSM
    |                                      |
    | --- vdc_SendVanish --------------->  |
    |    - dSUID: device dSUID             |
    |    (device going offline)            |
    |                                      |
```
If a managed device is going to be shut down or completely removed the vdC host will send a SendVanish message to vdSM

3) Device removal (vdSM triggered):

```
vDC Host                                vdSM
    |                                      |
    | <------------------ vdsm_Remove --   |
    |    - dSUID: device dSUID             |
    |    (device going offline)            |
    |                                      |
```

## Error Handling

### Generic Response

Errors are normally returned via GenericResponse mesaage as response in case of Errors:
```
vDC Host                                vdSM
    |                                      |
    | <------------- <Request> ---------   |
    |                - xxx                 |
    |                                      |
    | -------- GenericResponse -------->   |
    |        - ResultCode: resultCode      |
    |        - description: optional       |
    |                                      |                                                                                 ```    
```protobuf
enum ResultCode {
    ERR_OK = 0;                      // Success
    ERR_MESSAGE_UNKNOWN = 1;         // Unknown message type
    ERR_INCOMPATIBLE_API = 2;        // API version mismatch
    ERR_SERVICE_NOT_AVAILABLE = 3;   // Service unavailable
    ERR_INSUFFICIENT_STORAGE = 4;    // Out of storage
    ERR_FORBIDDEN = 5;               // Operation not allowed
    ERR_NOT_IMPLEMENTED = 6;         // Feature not implemented
    ERR_NO_CONTENT_FOR_ARRAY = 7;    // Array access error
    ERR_INVALID_VALUE_TYPE = 8;      // Wrong value type
    ERR_MISSING_SUBMESSAGE = 9;      // Required submessage missing
    ERR_MISSING_DATA = 10;           // Required data missing
    ERR_NOT_FOUND = 11;              // Resource not found
    ERR_NOT_AUTHORIZED = 12;         // Not authorized
}
```

### Error Handling Strategy

**As vDC Host**:

1. **Validate Requests**: Check message structure and data
2. **Return Appropriate Codes**: Use specific error codes
3. **Provide Descriptions**: Add helpful error messages
4. **Log Errors**: Track issues for debugging
5. **Recover Gracefully**: Don't crash on bad requests


### Common Error Scenarios

| Scenario | Code | Description |
|----------|------|-------------|
| Property doesn't exist | ERR_NOT_FOUND | "Property 'X' not found" |
| Read-only property | ERR_FORBIDDEN | "Property 'X' is read-only" |
| Invalid type | ERR_INVALID_VALUE_TYPE | "Expected string, got number" |
| Feature not supported | ERR_NOT_IMPLEMENTED | "Feature X not implemented" |
| Device offline | ERR_SERVICE_NOT_AVAILABLE | "Device not available" |

## Message ID Correlation

### Request/Response Matching

Use `message_id` to correlate requests with responses:

```python
# Sender (vdSM)
request = Message()
request.type = VDSM_REQUEST_GET_PROPERTY
request.message_id = 42  # Unique ID
send(request)

# Receiver (vDC Host)
response = Message()
response.type = VDC_RESPONSE_GET_PROPERTY
response.message_id = 42  # Same ID as request
send(response)
```

### Message ID Guidelines

- **Uniqueness**: Each request should have unique ID (per session)
- **Monotonic**: Typically increment by 1
- **Zero**: message_id = 0 often used for messages without response
- **Scope**: IDs are per-connection, not global

## Message Framing

### Length-Prefix Framing (Common)

Most implementations use length-prefix framing:

```
[4 bytes: length][N bytes: protobuf message]
[4 bytes: length][N bytes: protobuf message]
...
```

Example implementation:
```python
def send_message(sock, message):
    """Send a protobuf message with length prefix"""
    data = message.SerializeToString()
    length = len(data)
    sock.sendall(struct.pack('<I', length))  # 4 bytes, little-endian
    sock.sendall(data)

def recv_message(sock):
    """Receive a protobuf message with length prefix"""
    length_data = sock.recv(4)
    if not length_data:
        return None
    length = struct.unpack('<I', length_data)[0]
    data = b''
    while len(data) < length:
        chunk = sock.recv(length - len(data))
        if not chunk:
            raise ConnectionError("Connection closed")
        data += chunk
    return Message.FromString(data)
```

### Alternative Framing

Some implementations may use:
- Delimiter-based (less common)
- Fixed-size headers
- Other framing protocols

Check your specific dSS version documentation.

## API Versions

### Version Negotiation

API version is specified in the Hello handshake:

```protobuf
vdsm_RequestHello {
    api_version = 3  // Current version
}
```

### Version History

- **v1**: Initial version
- **v2**: Added dim channel support
- **v2c**: Added generic request, device events
- **v3**: Added channel ID support

### Version Compatibility

vDC hosts should:
1. Support the requested API version
2. Return ERR_INCOMPATIBLE_API if version not supported
3. Optionally support multiple versions

## Message Type Reference

### Session Management

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_REQUEST_HELLO | vdsm_RequestHello | vdSM → vDC | vdc_ResponseHello |
| VDC_RESPONSE_HELLO | vdc_ResponseHello | vDC → vdSM | N/A |
| VDSM_SEND_BYE | vdsm_SendBye | vdSM → vDC | None |

### Property Access

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_REQUEST_GET_PROPERTY | vdsm_RequestGetProperty | vdSM → vDC | vdc_ResponseGetProperty |
| VDC_RESPONSE_GET_PROPERTY | vdc_ResponseGetProperty | vDC → vdSM | N/A |
| VDSM_REQUEST_SET_PROPERTY | vdsm_RequestSetProperty | vdSM → vDC | GenericResponse |
| VDC_SEND_PUSH_PROPERTY | vdc_SendPushProperty | vDC → vdSM | Error only |
| VDC_SEND_PUSH_NOTIFICATION | vdc_SendPushNotification | vDC → vdSM | Error only |

### Device Lifecycle

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDC_SEND_ANNOUNCE_VDC | vdc_SendAnnounceVdc | vDC → vdSM | Error only |
| VDC_SEND_ANNOUNCE_DEVICE | vdc_SendAnnounceDevice | vDC → vdSM | Error only |
| VDC_SEND_VANISH | vdc_SendVanish | vDC → vdSM | Error only |
| VDSM_SEND_REMOVE | vdsm_SendRemove | vdSM → vDC | None |
| VDC_SEND_IDENTIFY | vdc_SendIdentify | vDC → vdSM | Error only |

### Keep-Alive

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_SEND_PING | vdsm_SendPing | vdSM → vDC | vdc_SendPong |
| VDC_SEND_PONG | vdc_SendPong | vDC → vdSM | N/A |

### Notifications (Scene Operations)

All notifications: vdSM → vDC, no response expected

| Type | Message | Purpose |
|------|---------|---------|
| VDSM_NOTIFICATION_CALL_SCENE | vdsm_NotificationCallScene | Activate scene |
| VDSM_NOTIFICATION_SAVE_SCENE | vdsm_NotificationSaveScene | Save current state as scene |
| VDSM_NOTIFICATION_UNDO_SCENE | vdsm_NotificationUndoScene | Undo scene |
| VDSM_NOTIFICATION_SET_LOCAL_PRIO | vdsm_NotificationSetLocalPrio | Set local priority |
| VDSM_NOTIFICATION_CALL_MIN_SCENE | vdsm_NotificationCallMinScene | Call minimum scene |
| VDSM_NOTIFICATION_IDENTIFY | vdsm_NotificationIdentify | Identify device(s) |
| VDSM_NOTIFICATION_SET_CONTROL_VALUE | vdsm_NotificationSetControlValue | Set control value |
| VDSM_NOTIFICATION_DIM_CHANNEL | vdsm_NotificationDimChannel | Dim channel |
| VDSM_NOTIFICATION_SET_OUTPUT_CHANNEL_VALUE | vdsm_NotificationSetOutputChannelValue | Set channel value |

### Generic Request (API v2c+)

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_REQUEST_GENERIC_REQUEST | vdsm_RequestGenericRequest | vdSM → vDC | GenericResponse |

## Best Practices

### Connection Management

1. **Reconnection**: Handle connection loss, implement reconnection logic
2. **Timeouts**: Set appropriate read/write timeouts
3. **Keep-Alive**: Respond to ping promptly
4. **Clean Shutdown**: Handle BYE message properly

### Message Handling

1. **Validate Input**: Check all message fields before processing
2. **Type Safety**: Verify message type matches expected type
3. **Error Responses**: Always respond to requests (success or error)
4. **Logging**: Log all messages for debugging (at debug level)

### Performance

1. **Async I/O**: Use non-blocking sockets for better performance
2. **Message Queue**: Queue outgoing messages
3. **Batch Updates**: Combine multiple property updates when possible
4. **Caching**: Cache property values to avoid repeated lookups

### Security

1. **Input Validation**: Validate all incoming data
2. **Resource Limits**: Limit message size, connection count
3. **Authentication**: Consider implementing authentication (future)
4. **Encryption**: Consider TLS for sensitive deployments

## Debugging Tips

### Message Logging

Log messages in human-readable format:

```python
def log_message(direction, message):
    print(f"{direction} {message.type}")
    print(f"  message_id: {message.message_id}")
    if message.HasField('vdsm_request_get_property'):
        print(f"  GetProperty for: {message.vdsm_request_get_property.dSUID}")
    # ... etc
```

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| No response | Request times out | Check message_id matching |
| Wrong message type | Unexpected message | Verify Type enum value |
| Parse error | Protobuf decode fails | Check framing, byte order |
| Connection drops | Frequent reconnects | Implement ping/pong |
| Property not found | ERR_NOT_FOUND | Verify property tree structure |

### Testing Tools

- **Wireshark**: Monitor TCP traffic (with protobuf dissector)
- **protoc**: Validate .proto files
- **Unit Tests**: Test message encoding/decoding
- **Mock vdSM**: Create test client for development

## Summary

Key protocol points:

1. **Protobuf over TCP**: Binary protocol, efficient encoding
2. **Connection Lifecycle**: Discovery → Connect → Handshake → Announce → Operate
3. **Message Types**: Requests, responses, notifications, sends
4. **Error Handling**: GenericResponse with ResultCode
5. **Correlation**: message_id links requests and responses
6. **Keep-Alive**: Ping/pong maintains connection
7. **Framing**: Typically length-prefixed messages

## Next Steps

- [Property System](./04-property-system.md) - Deep dive into property trees
- [API Operations](./08-api-operations.md) - Detailed operation descriptions
- [Protocol Buffer Reference](./09-protobuf-reference.md) - Complete message reference
- [Implementation Guide](./10-implementation-guide.md) - Practical implementation

---

[← Back: Core Concepts](./02-core-concepts.md) | [Back to Index](./README.md) | [Next: Property System →](./04-property-system.md)
