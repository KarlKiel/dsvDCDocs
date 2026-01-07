# Device Operations

## Overview

This document describes how vDCs and devices are announced, operated, and removed within an established vDC session.

## Announcement Hierarchy

The announcement process follows a strict hierarchy:

1. **vDC Session Initialization** - Hello exchange
2. **vDC Announcement** - Announce logical vDCs
3. **Device Announcement** - Announce devices managed by each vDC

> **Important:** vDCs must be announced before their devices can be announced.

## vDC Announcement

After the vDC session is established, the vDC host must announce every logical vDC it hosts before announcing any devices.

### Announce vDC Method

**Method:** `announcevdc`

**Direction:** vDC host → vdSM

**Message:** `vdc_SendAnnounceVdc`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex chars) | The dSUID of the vDC to be announced |

#### Success Response

**Message:** `GenericResponse`

| Field | Value | Description |
|-------|-------|-------------|
| `code` | 0 | Success |

#### Error Responses

**Message:** `GenericResponse`

| Code | Description |
|------|-------------|
| `ERR_INSUFFICIENT_STORAGE` | vdSM cannot handle another vDC |

Other fields in error response:
- `description` - Explanation text for developers
- `errorType` - Kind of failure with implications for error recovery
- `userMessageToBeTranslated` - User-friendly message

### vDC Lifecycle

- **Persistence:** Unlike individual devices, logical vDCs **cannot vanish** during an established vDC session
- **Stability:** Once announced, a vDC remains available for the entire session
- **Re-announcement:** vDCs must be re-announced after session re-establishment

## Device Announcement

For every announced vDC, the vDC host must announce all devices managed by that vDC.

### Managed Devices

A device is considered "managed" by a vDC when:

- The vDC has reasonably reliable information that the device is connected
- The device is connectable from the vDC
- **Important:** Devices should be announced even if temporarily offline

### Announce Device Method

**Method:** `announcedevice`

**Direction:** vDC host → vdSM

**Message:** `vdc_SendAnnounceDevice`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex chars) | The dSUID of the device to be announced |
| `vdc_dSUID` | string (34 hex chars) | The dSUID of the logical vDC containing this device |

#### Success Response

**Message:** `GenericResponse`

| Field | Value | Description |
|-------|-------|-------------|
| `code` | 0 | Success |

#### Error Responses

**Message:** `GenericResponse`

| Code | Description |
|------|-------------|
| `ERR_INSUFFICIENT_STORAGE` | vdSM cannot handle another device |

Other error fields:
- `message` - Explanation text

### Announcement Order

```
vDC Session Established
    |
    v
Announce vDC #1
    |
    v
Announce Device #1.1 (belongs to vDC #1)
Announce Device #1.2 (belongs to vDC #1)
    |
    v
Announce vDC #2
    |
    v
Announce Device #2.1 (belongs to vDC #2)
Announce Device #2.2 (belongs to vDC #2)
```

## Device Operations

After a device has been announced, device-level methods can be invoked and notifications sent by either party.

### Available Operations

Once announced, devices support:

1. **Property Access:** Read and write device properties
2. **Actions:** Execute device-specific actions (scenes, dimming, etc.)
3. **Notifications:** Receive real-time updates
4. **Configuration:** Learn-in, authenticate, firmware update
5. **Presence Polling:** Check device availability

> See [Actions and Notifications](actions-notifications.md) and [Properties](properties.md) for details.

## Ending Device Operation

Devices can be removed from operation in two ways:

### 1. Vanish (Device-Initiated)

The vDC host sends a "Vanish" notification when a device has been:
- Physically disconnected
- Unlearned from the vDC
- Lost connection (e.g., wireless device out of range)

**Method:** `vanish`

**Direction:** vDC host → vdSM

**Message:** `vdc_SendVanish`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex chars) | The dSUID of the device that has vanished |

#### Response

**Message:** `GenericResponse`

| Field | Value | Description |
|-------|-------|-------------|
| `code` | 0 | Success |

### 2. Remove (vdSM-Initiated)

The vdSM calls "remove" to request device removal from the vDC.

**Method:** `remove`

**Direction:** vdSM → vDC host

**Message:** `vdsm_SendRemove`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex chars) | The dSUID of the device to be removed |

#### Success Response

**Message:** `GenericResponse`

| Field | Value | Description |
|-------|-------|-------------|
| `code` | 0 | Success - device removed |

#### Error Responses

**Message:** `GenericResponse`

| Code | Description |
|------|-------------|
| `ERR_FORBIDDEN` | vDC does not allow removal (device is verifiably connected and operable) |

Other error fields:
- `message` - Explanation text

### When to Allow Removal

The vDC should **allow** removal except when it has:
- 100% knowledge the device is actually connected
- Verification the device is currently operable

#### Important Consideration: Device Migration

Consider wireless devices that might be:
1. Carried away from their original vDC
2. Paired with a new vDC in the same installation
3. Still registered with the original vDC

In this case, the dSS/dSA will want to remove the device from the former vDC. **The vDC must not reject such deletion attempts.**

## Device Lifecycle

### Complete Device Lifecycle

```
┌─────────────────────┐
│   Not Announced     │
└──────────┬──────────┘
           │
           │ vdc_SendAnnounceDevice
           v
┌─────────────────────┐
│     Announced       │
│    (Operational)    │
└──────────┬──────────┘
           │
           ├─ Property Access
           ├─ Action Execution
           ├─ Notifications
           ├─ Configuration
           │
           ├─ vdc_SendVanish ────> [Vanished]
           │                            │
           └─ vdsm_SendRemove ──> [Removed]
                                        │
                                        v
                              ┌─────────────────────┐
                              │  Not Announced      │
                              │ (May be re-announced)│
                              └─────────────────────┘
```

## Identify Method

Both vDC hosts and individual devices can be asked to identify themselves physically.

### Identify Device

**Direction:** vdSM → vDC host

**Message:** `vdc_SendIdentify`

**Purpose:** Request the device to make itself physically identifiable (e.g., flash LED, beep)

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex chars) | The dSUID of the device or vDC host to identify |

#### Response

Expect response only in case of error (GenericResponse).

## Implementation Examples

### Announcing a vDC and Devices

```python
def announce_vdc_and_devices(session, vdc):
    # 1. Announce the vDC
    announce_vdc_msg = vdc_SendAnnounceVdc()
    announce_vdc_msg.dSUID = vdc.dsuid
    response = session.send_and_wait(announce_vdc_msg)
    
    if response.code != ERR_OK:
        log.error(f"Failed to announce vDC: {response.description}")
        return
    
    # 2. Announce all devices in this vDC
    for device in vdc.get_devices():
        announce_device_msg = vdc_SendAnnounceDevice()
        announce_device_msg.dSUID = device.dsuid
        announce_device_msg.vdc_dSUID = vdc.dsuid
        
        response = session.send_and_wait(announce_device_msg)
        
        if response.code != ERR_OK:
            log.error(f"Failed to announce device {device.dsuid}: {response.description}")
        else:
            log.info(f"Device {device.dsuid} announced successfully")
```

### Handling Device Vanish

```python
def on_device_disconnected(session, device):
    # Device has been physically disconnected
    vanish_msg = vdc_SendVanish()
    vanish_msg.dSUID = device.dsuid
    
    session.send_notification(vanish_msg)
    log.info(f"Device {device.dsuid} has vanished")
```

### Handling Remove Request

```python
def on_remove_request(remove_msg):
    device_dsuid = remove_msg.dSUID
    device = find_device(device_dsuid)
    
    if device is None:
        # Device not found - allow removal
        return GenericResponse(code=ERR_OK)
    
    if device.is_verifiably_connected():
        # Device is definitely connected - reject removal
        return GenericResponse(
            code=ERR_FORBIDDEN,
            message="Device is currently connected and operational"
        )
    else:
        # Device might have migrated - allow removal
        remove_device(device_dsuid)
        return GenericResponse(code=ERR_OK)
```

## Addressable Entities

The following entities have dSUIDs and can be addressed by device-level methods:

1. **vDC host** - The physical device hosting vDCs
2. **Logical vDCs** - Virtual device connectors
3. **Virtual devices (vdSDs)** - Individual devices

All addressable entities support:
- Named property access (read/write)
- Presence polling (ping)

> See [Properties](properties.md) for details on property access.

## Best Practices

### For vDC Host Implementers

1. **Announcement Order:**
   - Always announce vDCs before their devices
   - Announce all vDCs first, then all devices

2. **Device Management:**
   - Announce devices even if temporarily offline
   - Send vanish notifications promptly when devices disconnect
   - Be permissive with remove requests for wireless devices

3. **Error Handling:**
   - Handle `ERR_INSUFFICIENT_STORAGE` gracefully
   - Log all announcement failures

4. **Re-connection:**
   - Re-announce all vDCs and devices after session re-establishment
   - Maintain device state across reconnections

### For vdSM Implementers

1. **Storage Management:**
   - Monitor available storage for vDCs and devices
   - Return `ERR_INSUFFICIENT_STORAGE` when limits reached

2. **Device Tracking:**
   - Track announced vDCs and devices
   - Clear state on session termination
   - Handle vanish notifications promptly

3. **Remove Requests:**
   - Respect `ERR_FORBIDDEN` responses
   - Handle device migration scenarios

## Next Steps

- Learn about [Properties](properties.md) for accessing device properties
- Review [Actions and Notifications](actions-notifications.md) for device control
- Explore [Session Management](session-management.md) for connection lifecycle
