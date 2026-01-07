# Connection Management

## Overview

The vDC-API uses TCP socket connections with Protocol Buffer message framing to enable communication between the vdSM (client) and vDC host (server).

## Transport Protocol

### TCP Connection

- **Direction:** vdSM initiates connection to vDC host
- **Protocol:** TCP/IP
- **Port:** Configurable (typically announced via Avahi, e.g., 8444)
- **Lifecycle:** Connection lifetime defines the vDC session

### Message Framing

Each message transmitted over the TCP stream consists of two parts:

#### 1. Message Header (2 bytes)
- **Format:** 16-bit unsigned integer
- **Byte Order:** Network byte order (big-endian)
- **Content:** Length of the Protocol Buffer message payload
- **Maximum:** 16,384 bytes (16 KB)

#### 2. Message Payload
- **Format:** Binary Protocol Buffer message
- **Schema:** Defined in `genericVDC.proto`
- **Maximum Size:** 16 KB (enforced by header field size)

### Wire Format Example

```
+----------------+----------------+------------------+
| Length MSB     | Length LSB     | Protobuf Message |
| (1 byte)       | (1 byte)       | (n bytes)        |
+----------------+----------------+------------------+
|<-- 2 bytes --->|<-- Length bytes (max 16384) ---->|
```

## Discovery

The vDC-API includes automatic service discovery to simplify network configuration.

### Requirements

1. vDC hosts must be automatically discoverable by vdSMs
2. Discovery must work without user interface on the vDC host
3. Wrong connections (e.g., to a neighbor's vDC) must be preventable

### Solution: Avahi/Bonjour

vDC hosts use **Avahi** (GNU implementation of Apple's Bonjour/mDNS) for service announcement:

#### vDC Host Advertisement

vDC hosts announce their presence using the service type `_ds-vdc._tcp`:

**Example `/etc/avahi/services/ds-vdc.service`:**
```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">digitalSTROM vDC host on %h</name>
  <service protocol="ipv4">
    <type>_ds-vdc._tcp</type>
    <port>8444</port>
  </service>
</service-group>
```

#### vdSM Advertisement (Historical)

vdSMs can also announce themselves using the service type `_ds-vdsm._tcp`:

**Example `/etc/avahi/services/ds-vdsm.service`:**
```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">digitalSTROM vdSM on %h</name>
  <service protocol="ipv4">
    <type>_ds-vdsm._tcp</type>
    <port>8441</port>
    <txt-record>dSUID=198C033E330755E78015F97AD093DD1C00</txt-record>
  </service>
</service-group>
```

### Discovery Process

1. **Announcement:** vDC hosts announce their service via Avahi
2. **Discovery:** vdSMs monitor Avahi announcements for `_ds-vdc._tcp` services
3. **Connection:** vdSM connects to at least one discovered vDC host
4. **Validation:** vDC host may reject connection if already connected to another vdSM

### Whitelist Protection

To prevent wrong connections in complex environments:

- **vdSM Whitelist:** Optional list of allowed vDC hosts
- **Location:** Maintained on the dSS device
- **Behavior:**
  - If whitelist exists: Only listed vDCs can be connected
  - If no whitelist: Automatic connection to any discovered vDC
- **Use Cases:**
  - Small installations: Fully automatic (no whitelist)
  - Large setups: Whitelists prevent conflicts in showrooms or developer environments

### Service Naming

- Service types use the `ds-` prefix to avoid collisions
- Chosen to be unlikely to conflict with other services
- May be registered with IANA if needed (per RFC 6335)

### IPv6 Support

- Current implementation uses IPv4 (`protocol="ipv4"`)
- Future versions may support IPv6 (`protocol="any"`)

## Connection Management

### Connection Establishment

1. **Discovery:** vdSM discovers vDC host via Avahi
2. **TCP Connect:** vdSM initiates TCP connection to announced port
3. **Handshake:** Session initialization (see [Session Management](session-management.md))

### Connection Maintenance

- **Lifetime:** vdSM aims to keep connections active at all times
- **Reconnection:** If connection is lost, vdSM attempts to re-establish
- **Ping/Pong:** Keep-alive mechanism to detect connection issues

### Connection Termination

- **Graceful:** `vdsm_SendBye` message before closing socket
- **Ungraceful:** Network failure or process termination
- **Effect:** All session state is lost; new session required

## Implementation Notes

### For vDC Host Developers

1. **TCP Server:** Implement TCP server on chosen port
2. **Avahi Service:** Create and install service description file
3. **Message Framing:** Read 2-byte header, then read specified bytes
4. **Maximum Message Size:** Enforce 16 KB limit
5. **Connection Limit:** Consider limiting to one vdSM connection at a time

### For vdSM Developers

1. **Avahi Client:** Monitor for `_ds-vdc._tcp` services
2. **Whitelist:** Implement optional whitelist filtering
3. **TCP Client:** Connect to discovered vDC hosts
4. **Message Framing:** Send 2-byte length header before each message
5. **Reconnection:** Implement automatic reconnection on failure

### Code Example (Pseudo-code)

#### Sending a Message

```python
def send_message(socket, protobuf_message):
    # Serialize the Protocol Buffer message
    payload = protobuf_message.SerializeToString()
    
    # Check size limit
    if len(payload) > 16384:
        raise ValueError("Message too large")
    
    # Create 2-byte header (big-endian)
    header = struct.pack('!H', len(payload))
    
    # Send header + payload
    socket.sendall(header + payload)
```

#### Receiving a Message

```python
def receive_message(socket):
    # Read 2-byte header
    header = socket.recv(2)
    if len(header) < 2:
        raise ConnectionError("Connection closed")
    
    # Parse length (big-endian)
    length = struct.unpack('!H', header)[0]
    
    # Validate length
    if length > 16384:
        raise ValueError("Message too large")
    
    # Read payload
    payload = b''
    while len(payload) < length:
        chunk = socket.recv(length - len(payload))
        if not chunk:
            raise ConnectionError("Connection closed")
        payload += chunk
    
    # Deserialize Protocol Buffer message
    message = Message()
    message.ParseFromString(payload)
    return message
```

## Error Handling

### Network Errors

- **Connection Refused:** vDC host not running or port blocked
- **Connection Reset:** vDC host terminated connection
- **Timeout:** Network issues or unresponsive vDC host

### Protocol Errors

- **Message Too Large:** Reject messages > 16 KB
- **Invalid Protobuf:** Log error and close connection
- **Version Mismatch:** Return `ERR_INCOMPATIBLE_API`

## Next Steps

- Learn about [Session Management](session-management.md) for the connection lifecycle
- Understand [API Basics](api-basics.md) for message structure
- Review [Error Handling](error-handling.md) for proper error recovery
