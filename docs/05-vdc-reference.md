# vDC Reference

## Overview

This document provides a complete reference for Virtual Device Connector (vDC) level properties, capabilities, and operations.

For implementation details, see [Implementation Guide](./10-implementation-guide.md).  
For property system basics, see [Property System](./04-property-system.md).

## vDC Properties

### Common Properties

All vDCs must provide these common properties (when querying with the vDC's dSUID):

| Property Path | Type | R/W | Description |
|---------------|------|-----|-------------|
| `/name` | string | R/W | Human-readable vDC name |
| `/dSUID` | string | R | Unique vDC identifier |
| `/modelName` | string | R | vDC model/type name |
| `/modelVersion` | string | R | vDC version |
| `/vendorName` | string | R | Vendor/manufacturer |
| `/oemModelGUID` | string | R | OEM model identifier |
| `/configURL` | string | R | Configuration web UI URL (optional) |
| `/iconName` | string | R | Icon identifier |
| `/implementationId` | string | R | Implementation identifier |
| `/zoneID` | uint | R/W | Default zone for the vDC |

### vDC Capabilities

The `/capabilities` container describes what the vDC supports:

| Property Path | Type | Description |
|---------------|------|-------------|
| `/capabilities/metering` | bool | Supports power/energy metering |
| `/capabilities/identification` | bool | Can identify vDC host hardware |
| `/capabilities/dynamicDefinitions` | bool | Supports dynamic device definitions |

**Example vDC Properties**:
```json
{
  "name": "Philips Hue vDC",
  "dSUID": "ABCD1234567890ABCD1234567890ABCD00",
  "modelName": "HueVdc",
  "modelVersion": "2.1.0",
  "vendorName": "MyCompany",
  "oemModelGUID": "hue-vdc-v2",
  "implementationId": "x-mycompany-huevdc",
  "zoneID": 0,
  "capabilities": {
    "metering": false,
    "identification": false,
    "dynamicDefinitions": false
  }
}
```

## vDC Lifecycle

### 1. vDC Announcement

After session establishment, announce the vDC:

```protobuf
vdc_SendAnnounceVdc {
    dSUID: "VDC_DSUID"
}
```

**When to announce**:
- Once per session after Hello handshake
- When a new vDC is added dynamically

### 2. Property Queries

After announcement, vdSM will query vDC properties:

**Typical query** (sent to vDC's dSUID):
```json
{
  "query": [
    { "name": "name" },
    { "name": "modelName" },
    { "name": "capabilities" }
  ]
}
```

**Expected response**: Requested vDC properties

### 3. Device Management

vDC manages its devices:
- Announce devices under this vDC
- Handle device operations
- Route notifications to devices

### 4. vDC Removal

vDCs typically persist for the session lifetime. If a vDC needs to be removed dynamically, all its devices should first send `Vanish` messages.

## vDC Types by Technology

### Integration vDCs

Integrate external device ecosystems:

**Examples**:
- **Hue vDC**: Philips Hue lights
- **Sonos vDC**: Sonos speakers
- **MQTT vDC**: MQTT-connected devices
- **HTTP vDC**: HTTP/REST API devices

**Characteristics**:
- One vDC per technology/brand
- Manages multiple devices
- Handles technology-specific protocol

### Protocol Bridge vDCs

Bridge other protocols to digitalSTROM:

**Examples**:
- **KNX vDC**: KNX building automation
- **Modbus vDC**: Modbus devices
- **BACnet vDC**: BACnet building systems

### Virtual/Simulation vDCs

For testing and simulation:

**Examples**:
- **Test vDC**: Mock devices for testing
- **Simulation vDC**: Simulated devices

## Best Practices

### Naming

- Use descriptive vDC names
- Include technology/brand name
- Version in modelVersion

**Good**: "Philips Hue Integration v2.1"  
**Bad**: "vDC1"

### dSUID Generation

Generate stable dSUIDs:

```python
import uuid

VDC_NAMESPACE = uuid.UUID('your-namespace-uuid')
vdc_id = f"vdc-{technology}-{instance}"
vdc_uuid = uuid.uuid5(VDC_NAMESPACE, vdc_id)
vdc_dsuid = vdc_uuid.bytes + b'\x00'
```

### Capability Reporting

Report accurate capabilities:
- Set `metering: true` only if you provide power data
- Set `identification: true` if you can physically identify the host

### Multiple vDCs per Host

A single vDC host can host multiple vDCs:

```
vDC Host
├── Hue vDC (dSUID: AAA...)
│   └── Devices...
├── Sonos vDC (dSUID: BBB...)
│   └── Devices...
└── MQTT vDC (dSUID: CCC...)
    └── Devices...
```

**When to use multiple vDCs**:
- Different device technologies
- Different external systems
- Logical separation needed

**When to use single vDC**:
- All devices from same technology
- Unified management
- Simple integration

## Configuration

### Configuration URL

Optionally provide a web UI for configuration:

```json
{
  "configURL": "http://192.168.1.100:8080/config"
}
```

This allows users to configure the vDC through a web interface.

### Persistent Configuration

Store vDC configuration:
- vDC name (user-modifiable)
- Connection settings
- Feature flags
- Not: dSUID (must be stable)

## Error Scenarios

### vDC Not Available

If vDC becomes unavailable:

```python
# Return error for operations
response = GenericResponse()
response.code = ERR_SERVICE_NOT_AVAILABLE
response.description = "vDC temporarily unavailable"
```

### Property Access Errors

Handle property errors appropriately:

| Scenario | Error Code |
|----------|------------|
| Property doesn't exist | ERR_NOT_FOUND |
| Read-only property | ERR_FORBIDDEN |
| Invalid value | ERR_INVALID_VALUE_TYPE |

## Related Documents

- [vdSD Reference](./06-vdsd-reference.md) - Device level properties
- [Property System](./04-property-system.md) - Property tree details
- [Implementation Guide](./10-implementation-guide.md) - Implementation examples

---

[Back to Documentation Index](./README.md)
