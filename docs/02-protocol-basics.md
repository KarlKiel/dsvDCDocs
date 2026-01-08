# Protocol Basics

## Protocol Basics

The vDC API uses a binary protocol based on **Protocol Buffers (protobuf)** over TCP connections.

### Transport Layer

- **Protocol**: TCP
- **Default Port**: 8440 (configurable via service announcement)
- **Connection**: Initiated by vdSM, accepted by vDC host
- **Encoding**: Protobuf binary messages
- **Framing**: Implementation-specific (often length-prefixed)

The **Protocol Buffers (protobuf)** is used for message encoding:
Protocol Buffers is a language-neutral, platform-neutral mechanism for serializing structured data:

- **Binary encoding**: Efficient on-the-wire representation
- **Schema-based**: Messages defined in `.proto` files
- **Strongly typed**: Type safety with automatic validation
- **Version-tolerant**: New fields can be added without breaking compatibility

## Message Structure

### Base Message Type

All messages use the `Message` wrapper:

```protobuf
message Message {
    required Type type = 1 [ default = GENERIC_RESPONSE ];
    optional uint32 message_id = 2 [ default = 0 ];
    optional GenericResponse generic_response = 3;
    
    // Specific message types (one will be set based on 'type')
    optional vdsm_RequestHello vdsm_request_hello = 100;
    optional vdc_ResponseHello vdc_response_hello = 101;
    // ... (many more message types)
}
```

### Message Components

1. **type**: Enum identifying the message type
2. **message_id**: Correlation ID for request/response matching
3. **Payload**: One of the optional message fields based on type

### Message Direction

Messages are prefixed to indicate direction:

- **vdsm_**: Message sent by vdSM (dSS) to vDC host
- **vdc_**: Message sent by vDC host to vdSM (dSS)

### Message Categories

Messages fall into several categories:

| Category | Description | Examples |
|----------|-------------|----------|
| **Request/Response** | Expects specific response |
| **Request/GenericResponse** | Expects GenericResponse | 
| **Send** | May send error response only | 
| **Notification** | No response expected | 



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

After handshake, announce all vDCs:

```
vDC Host                                vdSM
    |                                      |
    | -- vdc_SendAnnounceVdc ---------->   |
    |    - dSUID: vDC #1 dSUID             |
    |                                      |
    | -- vdc_SendAnnounceVdc ---------->   |
    |    - dSUID: vDC #2 dSUID             |
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
```
For every hosted vDC, vDC host will send a SendAnnounceDevice message for every managed device of the respective vDC

### 6. Operational Phase

During the Operational Phase the vdSM and vDC host exchange the regular operational messages: property reads/writes, pushes, pings, actions/notifications (scenes, dimming, identify, etc.). Below are diagrams for all operational messages (based on the vDC API reference). The diagrams follow the same style as earlier lifecycle diagrams and show direction and key payload fields.

Note: "vdsm_" prefix indicates a message sent by the vdSM (dSS). "vdc_" prefix indicates a message sent by the vDC host.

1) Get Property (Request/Response)

```
vDC Host                                vdSM
    |                                      |
    | <-------- vdsm_RequestGetProperty -- |
    |   - dSUID: target entity dSUID       |
    |   - query: (property tree structure) |
    |                                      |
    | --- vdc_ResponseGetProperty ------>  |
    |   - properties: (prop. tree struct.) |
    |                                      |
```

- Type: VDSM_REQUEST_GET_PROPERTY → VDC_RESPONSE_GET_PROPERTY
- Correlate with message_id in request/response.

2) Set Property (Request → GenericResponse)

```
vDC Host                                vdSM
    |                                      |
    | <------- vdsm_RequestSetProperty --- |
    |   - dSUID: target entity dSUID       |
    |   - properties: (prop. tree struct.) |
    |                                      |
    | --- vdc_GenericResponse -----------> |
    |   - code: ERR_OK / error code        |
    |   - description: (optional)          |
    |                                      |
```

- Type: VDSM_REQUEST_SET_PROPERTY → GenericResponse on error or success.

3) Push Property (vDC → vdSM notification)

```
vDC Host                                vdSM
    |                                      |
    | -- vdc_SendPushProperty ---------->  |
    |   - dSUID: origin entity dSUID       |
    |   - properties: (prop. tree struct.) |
    |                                      |
    | (no response expected; error via GenericResponse only) |
```

- Message used by vDC host to notify vdSM of changed values.

4) Generic Request (vdSM → vDC host → GenericResponse)

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_RequestGenericRequest --- |
    |   - dSUID: target (optional)         |
    |   - request: (generic payload)       |
    |                                      |
    | --- vdc_GenericResponse -----------> |
    |   - code: ERR_OK / error code        |
    |   - description: (optional)          |
    |                                      |
```

- Type: VDSM_REQUEST_GENERIC_REQUEST → GenericResponse

5) Ping / Pong (Keep-Alive / Presence polling)

```
vDC Host                                vdSM
    |                                      |
    | <-------- vdsm_SendPing ----------   |
    |   - dSUID: target entity dSUID       |
    |                                      |
    | -- vdc_SendPong ----------------->   |
    |   - dSUID: responder entity dSUID    |
    |                                      |
```

- Ping from vdSM, Pong from vDC host. Use for presence polling and keep-alive.

6) Call Scene (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_NotificationCallScene --- |
    |   - dSUID: one or many device dSUIDs |
    |   - scene: scene number              |
    |   - force: boolean                   |
    |   - group / zoneID: optional         |
    |                                      |
    | (no response expected)               |
```

7) Save Scene (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_NotificationSaveScene --- |
    |   - dSUID: one or many device dSUIDs |
    |   - scene: scene number              |
    |   - group / zoneID: optional         |
    |                                      |
    | (no response expected)               |
```

8) Undo Scene (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <---- vdsm_NotificationUndoScene --- |
    |   - dSUID: one or many device dSUIDs |
    |   - scene: scene number to undo      |
    |   - group / zoneID: optional         |
    |                                      |
    | (no response expected)               |
```

9) Set Local Priority (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationSetLocalPrio ---|
    |   - dSUID: target dSUID(s)           |
    |   - scene: scene to check            |
    |   - group / zoneID: optional         |
    |                                      |
    | (no response expected)               |
```

10) Dim Channel (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationDimChannel -----|
    |   - dSUID: device dSUID(s)           |
    |   - channel: channel index           |
    |   - mode: 1 / -1 / 0                 |
    |   - area: area restriction           |
    |                                      |
    | (no response expected)               |
```

11) Call Minimum Scene (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationCallMinScene ---|
    |   - dSUID: device dSUID(s)           |
    |   - scene: min scene number          |
    |   - group / zoneID: optional         |
    |                                      |
    | (no response expected)               |
```

12) Identify (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationIdentify -------|
    |   - dSUID: device dSUID(s)           |
    |   - parameters: (optional)           |
    |                                      |
    | (no response expected)               |
```

13) Set Control Value (vdSM → vDC host, notification)

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationSetControlValue -|
    |   - dSUID: device dSUID(s)           |
    |   - control: control identifier      |
    |   - value: value to set              |
    |                                      |
    | (no response expected)               |
```

14) Set Output Channel Value (vdSM → vDC host, notification/action)

```
vDC Host                                vdSM
    |                                      |
    | <- vdsm_NotificationSetOutputChannelValue -|
    |   - dSUID: device dSUID(s)                |
    |   - channel: output channel               |
    |   - value: target output value            |
    |                                      |
    | (no response expected; use GenericResponse on error) |
```

15) Push Notification (vDC → vdSM, informational push)

```
vDC Host                                vdSM
    |                                      |
    | -- vdc_SendPushNotification ------>   |
    |   - dSUID: origin entity dSUID        |
    |   - notification: payload/info        |
    |                                      |
    | (no response expected; error via GenericResponse only) |
```

16) Device Vanish (vDC → vdSM)

```
vDC Host                                vdSM
    |                                      |
    | -- vdc_SendVanish ---------------->   |
    |   - dSUID: vanished device dSUID      |
    |                                      |
    | (no response expected)                |
```

17) Remove Device (vdSM → vDC host → GenericResponse)

```
vDC Host                                vdSM
    |                                      |
    | <------ vdsm_SendRemove ------------ |
    |   - dSUID: device dSUID              |
    |                                      |
    | --- vdc_GenericResponse -----------> |
    |   - code: ERR_OK / error code        |
    |                                      |
```

Notes:
- For request/response messages use message_id for correlation (unique per session).
- Notifications typically don't expect a response; when an error must be reported the receiver can send a GenericResponse (implementation-specific handling).
- Where the API reference explicitly defines a response type, follow that (e.g. getProperty → ResponseGetProperty, setProperty → GenericResponse, getProperty error → GenericResponse).

### 7. Operational Phase — quick mapping to vDC API message names

- Property read: vdsm_RequestGetProperty ↔ vdc_ResponseGetProperty
- Property write: vdsm_RequestSetProperty → GenericResponse
- Property push: vdc_SendPushProperty → (no response)
- Ping/pong: vdsm_SendPing ↔ vdc_SendPong
- Actions/notifications: vdsm_NotificationCallScene, vdsm_NotificationSaveScene, vdsm_NotificationUndoScene, vdsm_NotificationSetLocalPrio, vdsm_NotificationDimChannel, vdsm_NotificationCallMinScene, vdsm_NotificationIdentify, vdsm_NotificationSetControlValue, vdsm_NotificationSetOutputChannelValue (all are notifications from vdSM to vDC host; no response expected unless error reporting is required)
- Generic requests: vdsm_RequestGenericRequest → GenericResponse
- Push notifications: vdc_SendPushNotification → (no response)

These diagrams and mappings are created to match the operational message set described in the vDC-API reference (vDC-API.md). They should be used as a quick visual reference when implementing the Operational Phase logic or when writing integration tests / mock clients.

### 7. Device Announcement / Operational Recall

vdC host announces devices and then normal operational messaging (see diagrams above). Use message_id for request/response correlation and respect the notification behavior for actions/scene-related messages.

### 8. Keep-Alive

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
For connection verification vdSM will periodically send SendPing messages and expects immediate SendPong messages from vdC host

### 9. Session Termination

Clean shutdown:

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

Or device removal (vDC triggered):

```
vDC Host                                vdSM
    |                                      |
    | -- vdc_SendVanish --------------->   |
    |    - dSUID: device dSUID             |
    |    (device going offline)            |
    |                                      |
```
If a managed device is going to be shut down or completely removed the vdC host will send a SendVanish message to vdSM

Or device removal (vdSM triggered:

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

Many operations return a GenericResponse as Response in case of Errors:
```
vDC Host                                vdSM
    |                                      |
    | <------------- Hello/Bye/Vanish --   |
    | <------------------ GetX / SetX --   |
    | ---- Announce_X ----------------->   |
    | <--> Notifications <-->              |
    | <--> Scene operations <-->           |
    |                 |
    |                                      |
```
```protobuf
message GenericResponse {
    required ResultCode code = 1 [ default = ERR_OK ];
    optional string description = 2;
}
```

### Result Codes

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

**Example Error Response**:
```json
{
  "code": "ERR_NOT_FOUND",
  "description": "Property '/device/unknown' does not exist"
}
```

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
