
## Message Types

The API defines several categories of messages:

### Request-Response Messages
Messages that expect a specific response:
- `vdsm_RequestHello` / `vdc_ResponseHello`
- `vdsm_RequestGetProperty` / `vdc_ResponseGetProperty`

### Commands
Messages that expect only a generic response or error:
- `vdsm_RequestSetProperty`
- `vdsm_RequestGenericRequest` (API v2c+)

### Notifications
Messages that do not expect any response:
- `vdsm_NotificationCallScene`
- `vdsm_NotificationSaveScene`
- `vdsm_NotificationDimChannel`
- etc.

### Send Messages
Messages that expect a response only in case of error:
- `vdsm_SendPing` / `vdc_SendPong`
- `vdc_SendAnnounceDevice`
- `vdc_SendVanish`

## Message Identification

Each message can include a `message_id` field:
- Used to correlate requests with responses
- Assigned by the sender
- Echo-ed back in the corresponding response
- Optional for notifications (which don't expect responses)

## Data Types

### PropertyValue

The API uses a flexible property value system supporting multiple data types:

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

Properties are represented as hierarchical elements:

```protobuf
message PropertyElement {
    optional string name = 1;
    optional PropertyValue value = 2;
    repeated PropertyElement elements = 3;
}
```

This allows for:
- Simple key-value properties
- Nested object structures
- Array properties

## Device Identification

### dSUID (digitalSTROM Unique Identifier)

Every entity in the system has a unique identifier called a dSUID:
- **Format:** String representation
- **Purpose:** Uniquely identifies vDCs, vdSDs, and other entities
- **Persistence:** Should remain stable across reboots and reconnections

### Addressing

Messages are addressed using the dSUID field:
- Set to the target device's dSUID for device-specific operations
- Set to the vDC's dSUID for vDC-level operations
- Used in responses to identify the responding entity

## API Versioning

The API includes version negotiation during the handshake:
- Client sends `api_version` in `vdsm_RequestHello`
- Server should check compatibility
- Returns `ERR_INCOMPATIBLE_API` if version is not supported

### Version History
- **API v2:** Added `vdsm_NotificationDimChannel`
- **API v2c:** Added `vdsm_RequestGenericRequest` and device events in push notifications
- **API v3:** Added `channelId` for channel operations

## Next Steps

- Learn about [Connection Management](connection-management.md) for implementing the network protocol
- Understand [Error Handling](error-handling.md) for proper error recovery
- Explore [Session Management](session-management.md) for managing the connection lifecycle
