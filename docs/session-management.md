# Session Management

## Overview

A vDC session represents the connection between a vdSM (virtual digitalSTROM Meter) and a vDC host. The session lifecycle includes initialization, operation, and termination phases.

## Session Basics

### Key Principles

- **Session Definition:** A session represents the connection from a single vdSM to a single vDC host
- **Connection Mapping:** A session is identical to having an active TCP connection
- **Persistence:** The vdSM aims to keep vDC sessions active at all times
- **Reconnection:** If a session terminates for any reason, the vdSM must attempt to re-establish it
- **Exclusivity:** At any given time, a vdSM may have at most one session with a particular vDC host
- **Multiple Hosts:** A vdSM may simultaneously connect to multiple distinct vDC hosts

### Session Lifecycle Phases

1. **Initialization:** Handshake and version negotiation
2. **Operation:** vDC and device announcement, property access, notifications
3. **Termination:** Graceful or ungraceful session end

## Session Initialization

### Initialization Flow

1. **TCP Connection:** vdSM establishes TCP connection to vDC host
2. **Hello Exchange:** vdSM sends `vdsm_RequestHello` to vDC host
3. **Version Check:** vDC host validates API version compatibility
4. **Response:** vDC host responds with `vdc_ResponseHello` or error

### Hello Method

**Direction:** vdSM → vDC host

**Message:** `vdsm_RequestHello`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex chars) | dSUID of the vdSM |
| `api_version` | integer | vDC API version (current document describes version 2) |

#### Success Response

**Message:** `vdc_ResponseHello`

| Field | Type | Description |
|-------|------|-------------|
| `dSUID` | string (34 hex chars) | dSUID of the vDC host (for future addressability) |

#### Error Responses

**Message:** `GenericResponse`

| Code | Description |
|------|-------------|
| `ERR_SERVICE_NOT_AVAILABLE` | vDC host cannot accept connection (already connected to another vdSM) |
| `ERR_INCOMPATIBLE_API` | vdSM API version is not supported |

### API Version Compatibility

The vDC host should check the `api_version` provided by the vdSM:

```text
If api_version is not supported:
    Return ERR_INCOMPATIBLE_API
    vdSM should cease connection attempts
    
If api_version is supported:
    Continue with session initialization
```

### Connection States

```
[Disconnected]
    |
    | vdSM connects via TCP
    v
[TCP Connected]
    |
    | vdSM sends vdsm_RequestHello
    v
[Handshaking]
    |
    |-- ERR_INCOMPATIBLE_API --> [Failed: Incompatible]
    |-- ERR_SERVICE_NOT_AVAILABLE --> [Failed: In Use]
    |-- vdc_ResponseHello --> [Session Initialized]
```

## Session Operation

### Operation Phase Activities

Once the session is initialized, the vDC host proceeds with:

1. **vDC Announcement:** Announce all logical vDCs hosted
2. **Device Announcement:** Announce all devices managed by each vDC
3. **Property Access:** Handle property read/write requests
4. **Notifications:** Send and receive notifications
5. **Keep-Alive:** Maintain presence through ping/pong

### Keep-Alive Mechanism

To prevent implicit session termination, regular communication must occur:

- **Mechanism:** Ping/Pong messages
- **Purpose:** Detect connection failures and maintain session
- **Frequency:** Regular intervals (implementation-dependent)

#### Ping/Pong Messages

**Ping (vdSM → vDC host):**
```protobuf
message vdsm_SendPing {
    optional string dSUID = 1;  // dSUID of the vDC host
}
```

**Pong (vDC host → vdSM):**
```protobuf
message vdc_SendPong {
    optional string dSUID = 1;  // dSUID of the vDC host
}
```

**Recommended Interval:** 30-60 seconds of inactivity

## Session Termination

### Explicit Termination

The session can be explicitly terminated using the `Bye` command.

**Method:** `bye`

**Direction:** vdSM → vDC host

**Message:** `vdsm_SendBye`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex chars) | dSUID of the vDC host |

#### Response

**Message:** `GenericResponse`

| Field | Value | Description |
|-------|-------|-------------|
| `code` | 0 | Success |

### Implicit Termination

The session is implicitly terminated when:

1. **New Hello:** A `vdsm_RequestHello` command is received (starts a new session)
2. **Connection Closed:** The TCP connection is closed by either party
3. **Network Failure:** Connection lost due to network issues

### Termination Effects

When a session terminates:

- **State Loss:** All session state is lost
- **vDC State:** Announced vDCs are forgotten
- **Device State:** Announced devices are forgotten
- **Reconnection Required:** New session must be established to resume operation

## Session State Diagram

```
┌─────────────────┐
│  Disconnected   │
└────────┬────────┘
         │ TCP Connect
         v
┌─────────────────┐
│ TCP Connected   │
└────────┬────────┘
         │ Send Hello
         v
┌─────────────────┐
│  Handshaking    │
└────────┬────────┘
         │
         ├─ Success ─────────> [Session Initialized]
         │                             │
         ├─ ERR_INCOMPATIBLE_API ───> [Failed]
         │                             │
         └─ ERR_SERVICE_NOT_AVAILABLE─> [Failed]
                                       │
                                       v
                              ┌─────────────────┐
                              │ Session Active  │<─┐
                              │  (Operation)    │  │
                              └────────┬────────┘  │
                                       │           │
                                       ├─ Ping/Pong
                                       ├─ Announce vDC
                                       ├─ Announce Device
                                       ├─ Property Access
                                       ├─ Notifications
                                       │
                                       ├─ Bye ───────> [Disconnected]
                                       ├─ Connection Lost ──> [Disconnected]
                                       └─ New Hello ────> [Handshaking]
```

## Implementation Guidelines

### For vDC Host Implementers

1. **Version Support:**
   - Clearly document supported API versions
   - Reject incompatible versions with appropriate error

2. **Connection Limits:**
   - Implement single vdSM connection limitation
   - Return `ERR_SERVICE_NOT_AVAILABLE` for additional connection attempts

3. **Keep-Alive:**
   - Respond promptly to ping messages with pong
   - Track last communication time
   - Consider connection dead after timeout

4. **State Management:**
   - Clear all session state on termination
   - Be prepared to re-announce all vDCs and devices on reconnection

### For vdSM Implementers

1. **Automatic Reconnection:**
   - Retry connection on failure
   - Use exponential backoff for retry attempts
   - Only cease retries for `ERR_INCOMPATIBLE_API`

2. **Version Negotiation:**
   - Send correct `api_version` in Hello
   - Handle version mismatch gracefully

3. **Keep-Alive:**
   - Send ping at regular intervals during inactivity
   - Monitor for pong responses
   - Detect dead connections

4. **Graceful Shutdown:**
   - Send `Bye` before closing connection when possible
   - Handle ungraceful shutdowns

## Example Session Flow

### Successful Session

```
vdSM                                      vDC host
  |                                           |
  |-- TCP Connect --------------------------->|
  |                                           |
  |-- vdsm_RequestHello ----------------------|
  |    (dSUID, api_version=2)                 |
  |                                           |
  |<- vdc_ResponseHello -----------------------|
  |    (dSUID)                                |
  |                                           |
  |<- vdc_SendAnnounceVdc ---------------------|
  |<- vdc_SendAnnounceDevice ------------------|
  |<- vdc_SendAnnounceDevice ------------------|
  |                                           |
  |-- vdsm_SendPing -------------------------->|
  |<- vdc_SendPong ----------------------------|
  |                                           |
  |   ... normal operation ...                |
  |                                           |
  |-- vdsm_SendBye --------------------------->|
  |<- GenericResponse (code=0) ----------------|
  |                                           |
  |-- TCP Close ------------------------------>|
```

### Failed Session (Already Connected)

```
vdSM                                      vDC host
  |                                           |
  |-- TCP Connect --------------------------->|
  |                                           |
  |-- vdsm_RequestHello ----------------------|
  |    (dSUID, api_version=2)                 |
  |                                           |
  |<- GenericResponse -------------------------|
  |    (code=ERR_SERVICE_NOT_AVAILABLE)       |
  |                                           |
  |-- TCP Close (wait and retry later) ------>|
```

### Failed Session (Incompatible Version)

```
vdSM                                      vDC host
  |                                           |
  |-- TCP Connect --------------------------->|
  |                                           |
  |-- vdsm_RequestHello ----------------------|
  |    (dSUID, api_version=99)                |
  |                                           |
  |<- GenericResponse -------------------------|
  |    (code=ERR_INCOMPATIBLE_API)            |
  |                                           |
  |-- TCP Close (do not retry) -------------->|
```

## Next Steps

- Learn about [Device Operations](device-operations.md) for announcing and managing devices
- Review [API Basics](api-basics.md) for message structure
- Explore [Properties](properties.md) for property system details
