# Properties

## Overview

The vDC-API uses a hierarchical property system for configuration and state management. Properties provide a flexible way to read and write device configuration, query device state, and access metadata.

## Property System Basics

### Addressable Entities

Addressable entities are items within the vDC host that have their own dSUID:

- **vDC host** - The physical device hosting vDCs
- **Logical vDCs** - Virtual device connectors
- **Virtual devices (vdSDs)** - Individual devices

Each addressable entity can have named properties that can be read and written by the vdSM.

### Property Structure

Properties use a hierarchical tree structure with the following components:

#### PropertyElement

```protobuf
message PropertyElement {
    optional string name = 1;           // Property name/key
    optional PropertyValue value = 2;   // Property value
    repeated PropertyElement elements = 3;  // Nested properties
}
```

#### PropertyValue

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

### Supported Value Types

- **integer** - Signed or unsigned integers
- **double** - Floating-point numbers
- **boolean** - True/false values
- **string** - UTF-8 encoded text
- **binary bytes** - Binary data
- **nested elements** - Lists or objects containing more properties

> **Recommendation:** Keep nesting levels as low as possible for simplicity.

## Property Categories

### System Properties

Properties defined in the digitalSTROM specifications with standard name, type, and behavior. Conforming implementations **must** support these.

### Custom Properties

Implementation-specific properties extending functionality beyond standard dS requirements.

**Naming Convention:**
- Must be prefixed with `x-`
- Should include vendor identifier
- Example: `x-abc-customSetting` (from company "Abc Inc.")

## Common Properties

All addressable entities must support the following common properties:

| Property | Access | Type | Description |
|----------|--------|------|-------------|
| `dSUID` | read-only | string (34 hex) | The dSUID of the entity |
| `displayId` | read-only | string | Human-readable identification printed on physical device |
| `type` | read-only | string | Type of entity: "vdSD", "vDC", "vDChost", or "vdSM" |
| `model` | read-only | string | Human-readable model string |
| `modelVersion` | read-only | string | Model version string |
| `modelUID` | read-only | string | digitalSTROM system unique ID for the functional model |

### Entity Types

- **`vdSD`** - Virtual dS device
- **`vDC`** - Logical vDC
- **`vDChost`** - The vDC host
- **`vdSM`** - A vdSM

### modelUID vs hardwareModelGuid

- **Same modelUID:** Functionally identical entities with exactly the same dS functionality
- **Same hardwareModelGuid:** Same hardware, potentially different dS mapping
- **Example:** Two identical hardware input devices may have:
  - Same `hardwareModelGuid`
  - Different `modelUID` if one is mapped as button, other as binaryInput

## Reading Properties

### GetProperty Method

**Method:** `getProperty`

**Direction:** vdSM → vDC host

**Message:** `vdsm_RequestGetProperty`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex) | dSUID of entity to read properties from |
| `query` | PropertyElement[] | Property tree structure specifying properties to return (no values) |

#### Query Structure

The `query` parameter uses a property tree with **names only** (no values):

- **Specific property:** Set `name` to the property name
- **Wildcard (all elements):** Set `name` to empty string
- **All deeper levels:** Use empty name as last element in branch

#### Response

**Message:** `vdc_ResponseGetProperty`

| Field | Type | Description |
|-------|------|-------------|
| `properties` | PropertyElement[] | Property tree with values |

#### Response Behavior

- Same tree structure as query
- **Wildcards expand:** Response includes all matching substructures
- **Unknown properties:** Not included in response (no error)
- **Properties without values:** Return explicit NULL value
- **No error** for non-existent properties (see Error Handling below)

#### Error Responses

**Message:** `GenericResponse`

| Code | Description |
|------|-------------|
| `ERR_FORBIDDEN` | Property exists but cannot be read (write-only, uncommon) |

### Query Examples

#### Example 1: Read Single Property

**Query:**
```json
{
  "query": [
    { "name": "model" }
  ]
}
```

**Response:**
```json
{
  "properties": [
    { "name": "model", "value": { "v_string": "SmartLight Pro" } }
  ]
}
```

#### Example 2: Read All Properties (Wildcard)

**Query:**
```json
{
  "query": [
    { "name": "" }  // Empty name = wildcard
  ]
}
```

**Response:** All available properties returned

#### Example 3: Read Nested Properties

**Query:**
```json
{
  "query": [
    { 
      "name": "inputs",
      "elements": [
        { "name": "0" },
        { "name": "1" }
      ]
    }
  ]
}
```

**Response:** Returns input 0 and input 1 configurations

#### Example 4: Read All Inputs

**Query:**
```json
{
  "query": [
    { 
      "name": "inputs",
      "elements": [
        { "name": "" }  // All inputs
      ]
    }
  ]
}
```

**Response:** All inputs with all their properties

## Writing Properties

### SetProperty Method

**Method:** `setProperty`

**Direction:** vdSM → vDC host

**Message:** `vdsm_RequestSetProperty`

#### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex) | dSUID of entity to write properties to |
| `properties` | PropertyElement[] | Property tree with names and values |

#### Property Structure for Writing

- **Specific property:** Set `name` and `value`
- **Wildcard (all elements):** Empty `name` applies same value to all elements

> **Important:** `channelState` must be changed using `setOutputChannelValue` action, not `setProperty`.

#### Response

**Message:** `GenericResponse`

| Field | Value | Description |
|-------|-------|-------------|
| `code` | 0 | Success |

#### Error Responses

**Message:** `GenericResponse`

| Code | Description |
|------|-------------|
| `ERR_INVALID_VALUE_TYPE` | Wrong data type provided |
| `ERR_FORBIDDEN` | Property is read-only |
| `ERR_NOT_FOUND` | Entity (device/vDC) not found |

#### Atomic Behavior

When writing multiple properties in one request:
- If **any** property write fails, the entire request returns an error
- **Some properties may already be stored** (in any order)
- Use **separate requests** for independent error status per property

### Write Examples

#### Example 1: Write Single Property

**Request:**
```json
{
  "dSUID": "199500440F80000000000F8001DEADBEEF",
  "properties": [
    { 
      "name": "name",
      "value": { "v_string": "Living Room Light" }
    }
  ]
}
```

#### Example 2: Write Multiple Properties

**Request:**
```json
{
  "dSUID": "199500440F80000000000F8001DEADBEEF",
  "properties": [
    { "name": "name", "value": { "v_string": "Kitchen Light" } },
    { "name": "zone", "value": { "v_uint64": 2 } }
  ]
}
```

#### Example 3: Write to All Elements (Wildcard)

**Request:**
```json
{
  "properties": [
    {
      "name": "outputs",
      "elements": [
        {
          "name": "",  // All outputs
          "elements": [
            { "name": "mode", "value": { "v_uint64": 1 } }
          ]
        }
      ]
    }
  ]
}
```

## Property Push Notifications

Some properties (especially button/input/sensor states) change within the device and can be reported to the dS system via push notifications.

### PushProperty Notification

**Notification:** `pushProperty` / `vdc_SendPushNotification`

**Direction:** vDC host → vdSM

**Message:** `vdc_SendPushNotification`

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `dSUID` | string (34 hex) | dSUID of entity originating the notification |
| `changedproperties` | PropertyElement[] | Property tree with changed values |
| `deviceevents` | PropertyElement[] | Device events (API v2c+) |

### Push Notification Example

**Button Press Notification:**
```json
{
  "dSUID": "199500440F80000000000F8001DEADBEEF",
  "changedproperties": [
    {
      "name": "buttonInputStates",
      "elements": [
        {
          "name": "0",
          "elements": [
            { "name": "value", "value": { "v_uint64": 1 } },
            { "name": "age", "value": { "v_double": 0.0 } }
          ]
        }
      ]
    }
  ]
}
```

## Property Access Patterns

### Pattern 1: Discovery

**Goal:** Discover what properties are available

**Approach:** Use wildcard queries

```python
def discover_properties(session, dsuid):
    query = PropertyElement(name="")  # Wildcard
    response = session.getProperty(dsuid, [query])
    return response.properties
```

### Pattern 2: Monitoring

**Goal:** Monitor changing properties

**Approach:** Subscribe to push notifications

```python
def handle_push_notification(notification):
    dsuid = notification.dSUID
    for prop in notification.changedproperties:
        log.info(f"Property {prop.name} changed on {dsuid}")
        update_cache(dsuid, prop)
```

### Pattern 3: Bulk Configuration

**Goal:** Configure multiple properties at once

**Approach:** Use single setProperty with multiple elements

```python
def configure_device(session, dsuid, config):
    properties = []
    for name, value in config.items():
        properties.append(PropertyElement(
            name=name,
            value=PropertyValue(v_string=value)
        ))
    
    session.setProperty(dsuid, properties)
```

## Device-Specific Properties

Different types of devices support different property sets:

### Button Inputs
- `buttonInputDescriptions` - Button configuration
- `buttonInputSettings` - Button settings
- `buttonInputStates` - Current button states

### Binary Inputs
- `binaryInputDescriptions` - Binary input configuration
- `binaryInputSettings` - Binary input settings
- `binaryInputStates` - Current binary input states

### Sensor Inputs
- `sensorInputDescriptions` - Sensor configuration
- `sensorInputSettings` - Sensor settings  
- `sensorInputStates` - Current sensor values

### Outputs and Channels
- `outputDescriptions` - Output configuration
- `outputSettings` - Output settings
- `outputStates` - Current output states
- `channelDescriptions` - Channel information
- `channelStates` - Current channel values

### Scenes
- `sceneDescriptions` - Scene metadata
- `scenes` - Scene values and configuration

> See the complete vDC-API-properties specification for detailed property schemas.

## Best Practices

### For vDC Implementers

1. **Support Common Properties:** Implement all required common properties
2. **Use Standard Names:** Follow naming conventions in specifications
3. **Custom Properties:** Prefix with `x-vendorname-`
4. **Push Updates:** Send push notifications for state changes
5. **UTF-8 Encoding:** Use UTF-8 for all string values
6. **Read-Only Properties:** Return `ERR_FORBIDDEN` for write attempts

### For vdSM Implementers

1. **Handle Missing Properties:** Don't error on non-existent properties in read
2. **Use Wildcards:** Leverage wildcards for discovery
3. **Cache Values:** Cache property values and update with push notifications
4. **Atomic Writes:** Be aware of partial write behavior
5. **Error Handling:** Check error codes and types

## Implementation Example

```python
class PropertyManager:
    def __init__(self, session):
        self.session = session
        self.cache = {}
    
    def get_property(self, dsuid, property_name):
        """Read a single property"""
        query = PropertyElement(name=property_name)
        response = self.session.getProperty(dsuid, [query])
        
        if response.properties:
            prop = response.properties[0]
            self.cache[(dsuid, property_name)] = prop.value
            return prop.value
        return None
    
    def set_property(self, dsuid, property_name, value):
        """Write a single property"""
        prop = PropertyElement(name=property_name, value=value)
        response = self.session.setProperty(dsuid, [prop])
        
        if response.code == ERR_OK:
            self.cache[(dsuid, property_name)] = value
        return response.code == ERR_OK
    
    def discover_all_properties(self, dsuid):
        """Discover all properties using wildcard"""
        query = PropertyElement(name="")  # Wildcard
        response = self.session.getProperty(dsuid, [query])
        return response.properties
    
    def on_push_notification(self, notification):
        """Handle property push notifications"""
        dsuid = notification.dSUID
        for prop in notification.changedproperties:
            self.update_cache(dsuid, prop)
    
    def update_cache(self, dsuid, property_element):
        """Recursively update cache from property tree"""
        if property_element.value:
            key = (dsuid, property_element.name)
            self.cache[key] = property_element.value
        
        for elem in property_element.elements:
            self.update_cache(dsuid, elem)
```

## Next Steps

- Review [Actions and Notifications](actions-notifications.md) for device control
- Explore [Device Operations](device-operations.md) for device management
- See the complete vDC-API-properties PDF for detailed property schemas
