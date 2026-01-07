# Property System - Understanding Property Trees

## Overview

The **Property System** is the core mechanism for accessing and manipulating configuration, state, and capabilities of vDCs and vdSDs in the digitalSTROM system. Understanding property trees is **essential** for implementing a vDC host.

This document provides a comprehensive guide to property trees, their structure, and how to work with them effectively.

## What is a Property Tree?

A property tree is a **hierarchical, JSON-like structure** that represents all accessible aspects of a vDC or vdSD. Think of it as a file system where:

- **Paths** navigate the hierarchy (e.g., `/device/name`)
- **Nodes** can be containers (objects) or values
- **Values** have types (string, number, boolean, etc.)
- **Arrays** can contain multiple elements

### Why Property Trees?

Property trees provide:

1. **Unified Access**: Single mechanism for all data access
2. **Discoverability**: Clients can explore available properties
3. **Type Safety**: Properties have defined types
4. **Hierarchical Organization**: Logical grouping of related properties
5. **Flexibility**: Easy to extend with new properties

## Property Element Structure

Properties are represented using the `PropertyElement` message defined in the Protocol Buffer specification:

```protobuf
message PropertyValue {
    optional bool v_bool = 1;
    optional uint64 v_uint64 = 2;
    optional int64 v_int64 = 3;
    optional double v_double = 4;
    optional string v_string = 5;
    optional bytes v_bytes = 6;
}

message PropertyElement {
    optional string name = 1;           // Property name
    optional PropertyValue value = 2;   // Property value (if leaf node)
    repeated PropertyElement elements = 3;  // Child elements (if container)
}
```

### Key Characteristics

- **name**: The property name (e.g., "device", "name", "dSUID")
- **value**: Set if this is a leaf node with an actual value
- **elements**: Set if this is a container with child properties

A PropertyElement is **either**:
- A **container** (has `elements`, no `value`) - like an object
- A **leaf** (has `value`, no `elements`) - like a field
- An **array element** (may have value or elements)

## Property Tree Structure

### Hierarchical Representation

The property tree structure differs depending on which entity (vDC or device) you are querying. Each entity has its own set of properties at the root level.

**Notation**:
- `propertyName` - Property element containing a value
- `[propertyName]` - Property element containing a list of child elements (container/array)

**vDC Level Property Tree** (when querying a vDC entity using the vDC's dSUID):

```
/ (root - all properties belong to the vDC)
├── dSUID (string)
├── displayId (string, optional)
├── type (string) = "vDC"
├── model (string)
├── modelVersion (string, optional)
├── modelUID (string)
├── hardwareVersion (string, optional)
├── hardwareGuid (string, optional)
├── hardwareModelGuid (string, optional)
├── vendorName (string, optional)
├── vendorGuid (string, optional)
├── oemGuid (string, optional)
├── oemModelGuid (string, optional)
├── configURL (string, optional)
├── deviceIcon16 (binary, optional)
├── deviceIconName (string, optional)
├── name (string, read/write)
├── deviceClass (string, optional)
├── deviceClassVersion (string, optional)
├── active (boolean, optional)
├── implementationId (string)
├── zoneID (uint64, read/write)
└── [capabilities]
    ├── metering (boolean)
    ├── identification (boolean)
    └── dynamicDefinitions (boolean)
```

**vdSD (Device) Level Property Tree** (when querying a device entity using the device's dSUID):

```
/ (root - all properties belong to the device)
├── dSUID (string)
├── displayId (string, optional)
├── type (string) = "vdSD"
├── model (string)
├── modelVersion (string, optional)
├── modelUID (string)
├── hardwareVersion (string, optional)
├── hardwareGuid (string, optional)
├── hardwareModelGuid (string, optional)
├── vendorName (string, optional)
├── vendorGuid (string, optional)
├── oemGuid (string, optional)
├── oemModelGuid (string, optional)
├── configURL (string, optional)
├── deviceIcon16 (binary, optional)
├── deviceIconName (string, optional)
├── name (string, read/write)
├── deviceClass (string, optional)
├── deviceClassVersion (string, optional)
├── active (boolean, optional)
├── primaryGroup (uint64)
├── zoneID (uint64, read/write)
│
├── [buttonInputDescriptions] (optional)
│   └── [0], [1], ... [N-1]
│       ├── name (string)
│       ├── dsIndex (uint64)
│       ├── supportsLocalKeyMode (boolean)
│       ├── buttonID (uint64, optional)
│       ├── buttonType (uint64)
│       └── buttonElementID (uint64)
│
├── [buttonInputSettings] (optional)
│   └── [0], [1], ... [N-1]
│       ├── group (uint64, read/write)
│       ├── function (uint64, read/write)
│       ├── mode (uint64, read/write)
│       ├── channel (uint64, read/write)
│       ├── setsLocalPriority (boolean, read/write)
│       └── callsPresent (boolean, read/write)
│
├── [buttonInputStates] (optional)
│   └── [0], [1], ... [N-1]
│       ├── value (boolean or NULL)
│       ├── clickType (uint64)
│       ├── age (double or NULL)
│       ├── error (uint64)
│       ├── actionId (uint64, optional - alternative to value/clickType)
│       └── actionMode (uint64, optional - alternative to value/clickType)
│
├── [binaryInputDescriptions] (optional)
│   └── [0], [1], ... [N-1]
│       ├── name (string)
│       ├── dsIndex (uint64)
│       ├── inputType (uint64)
│       └── inputUsage (uint64)
│
├── [binaryInputSettings] (optional)
│   └── [0], [1], ... [N-1]
│       ├── group (uint64, read/write)
│       ├── minPushInterval (uint64, read/write)
│       └── changesOnlyInterval (uint64, read/write)
│
├── [binaryInputStates] (optional)
│   └── [0], [1], ... [N-1]
│       ├── value (boolean or NULL)
│       ├── extendedValue (uint64 or NULL)
│       ├── age (double or NULL)
│       └── error (uint64)
│
├── [sensorDescriptions] (optional)
│   └── [0], [1], ... [N-1]
│       ├── name (string)
│       ├── dsIndex (uint64)
│       ├── sensorType (uint64)
│       ├── sensorUsage (uint64)
│       ├── min (double)
│       ├── max (double)
│       ├── resolution (double)
│       ├── updateInterval (double)
│       └── aliveSignInterval (double)
│
├── [sensorSettings] (optional)
│   └── [0], [1], ... [N-1]
│       └── group (uint64, read/write)
│
├── [sensorStates] (optional)
│   └── [0], [1], ... [N-1]
│       ├── value (double or NULL)
│       ├── age (double or NULL)
│       └── error (uint64)
│
├── [outputDescription] (optional, for devices with output)
│   ├── function (string)
│   └── outputUsage (uint64)
│
├── [outputSettings] (optional)
│   ├── mode (uint64, read/write)
│   └── ... (device-specific settings)
│
├── [outputState] (optional)
│   └── ... (device-specific state info)
│
├── [channelDescriptions] (optional, for devices with output)
│   └── [0], [1], ... [N-1]
│       ├── name (string)
│       ├── channelType (uint64)
│       ├── dsIndex (uint64)
│       ├── min (double)
│       ├── max (double)
│       └── resolution (double)
│
├── [channelSettings] (optional)
│   └── [0], [1], ... [N-1]
│       └── (currently no standard settings defined)
│
├── [channelStates] (optional)
│   └── [0], [1], ... [N-1]
│       ├── value (double)
│       └── age (double or NULL)
│
├── [scenes] (optional, for devices with output)
│   └── [0], [5], [17], ... (named by scene number)
│       └── [channels]
│           └── [0], [1], ... [N-1] (by channel index)
│               ├── value (double, read/write)
│               ├── dontCare (boolean, read/write)
│               └── automatic (boolean, read/write)
│
├── [deviceActionDescriptions] (optional)
│   └── [0], [1], ... [N-1]
│       └── ... (action description properties)
│
├── [customActions] (optional)
│   └── [0], [1], ... [N-1]
│       └── ... (custom action properties)
│
├── [deviceStateDescriptions] (optional)
│   └── [0], [1], ... [N-1]
│       └── ... (state description properties)
│
├── [deviceStates] (optional)
│   └── [0], [1], ... [N-1]
│       └── ... (state values)
│
├── [devicePropertyDescriptions] (optional)
│   └── [0], [1], ... [N-1]
│       └── ... (property descriptions)
│
├── [deviceProperties] (optional)
│   └── [0], [1], ... [N-1]
│       └── ... (property values)
│
└── [deviceEventDescriptions] (optional)
    └── [0], [1], ... [N-1]
        └── ... (event descriptions)
```

**Important Notes**:

1. **No "vdc" or "device" containers**: Properties are directly at the root level when querying an entity.
2. **dSUID in query**: You specify which entity to query using the dSUID parameter in GetProperty.
3. **Common properties**: Many properties (Section 2 in original API) are common to both vDC and vdSD entities.
4. **Optional properties**: Many properties are optional and may not be present in all implementations.
5. **Array indices**: Array elements are named by their index as strings: "0", "1", "2", etc.
6. **Scene naming**: Scene elements are named by scene number (e.g., "0", "5", "17"), not sequential indices.

### Property Paths

Properties are accessed using **path notation**. The paths depend on the entity being queried:

**When querying a vDC** (using vDC's dSUID):
- Root: `/`
- vDC name: `/name`
- vDC capabilities: `/capabilities/metering`

**When querying a device** (using device's dSUID):
- Root: `/`
- Device name: `/name`
- Button input: `/buttonInputDescriptions/0/name`
- Button state: `/buttonInputStates/0/value`
- Sensor value: `/sensorStates/2/value`
- Channel value: `/channelStates/0/value`
- Scene: `/scenes/5/name`

## Reading Properties

### GetProperty Request

To read properties, send a `vdsm_RequestGetProperty` message:

```protobuf
message vdsm_RequestGetProperty {
    optional string dSUID = 1;          // Target dSUID (vDC or device)
    repeated PropertyElement query = 2;  // Query specification
}
```

### Query Specification

The `query` field specifies **what to retrieve**:

#### Query Pattern 1: Request Specific Properties

Request specific properties by naming them:

```json
// Query for device name and dSUID
query: [
  {
    "name": "name"
  },
  {
    "name": "dSUID"
  }
]

// Response:
properties: [
  {
    "name": "name",
    "value": { "v_string": "Living Room Light" }
  },
  {
    "name": "dSUID",
    "value": { "v_string": "AB12CD34EF5678..." }
  }
]
```

#### Query Pattern 2: Request Nested Properties

Use nested PropertyElements to query hierarchical structures:

```json
// Query for vDC capabilities (when querying vDC entity)
query: [
  {
    "name": "capabilities",
    "elements": [
      { "name": "metering" },
      { "name": "identification" }
    ]
  }
]

// Response:
properties: [
  {
    "name": "capabilities",
    "elements": [
      {
        "name": "metering",
        "value": { "v_bool": false }
      },
      {
        "name": "identification",
        "value": { "v_bool": true }
      }
    ]
  }
]
```

#### Query Pattern 3: Request All Properties

Omit the `elements` field to request all child properties:

```json
// Query for all vDC capabilities (when querying vDC entity)
query: [
  {
    "name": "capabilities"
    // No elements field = get everything under capabilities
  }
]

// Response: All capability properties returned
properties: [
  {
    "name": "capabilities",
    "elements": [
      { "name": "metering", "value": { "v_bool": false } },
      { "name": "identification", "value": { "v_bool": true } },
      { "name": "dynamicDefinitions", "value": { "v_bool": false } }
    ]
  }
]
```

#### Query Pattern 4: Array Access

Access array elements by index (when querying device entity):

```json
// Query for first button input description
query: [
  {
    "name": "buttonInputDescriptions",
    "elements": [
      {
        "name": "0",  // Array index as string
        "elements": [
          { "name": "name" },
          { "name": "buttonType" }
        ]
      }
    ]
  }
]
```

### GetProperty Response

The response is a `vdc_ResponseGetProperty`:

```protobuf
message vdc_ResponseGetProperty {
    repeated PropertyElement properties = 1;
}
```

The `properties` field contains the requested data in the same hierarchical structure as the query.

## Writing Properties

### SetProperty Request

To write properties, send a `vdsm_RequestSetProperty` message:

```protobuf
message vdsm_RequestSetProperty {
    optional string dSUID = 1;             // Target dSUID
    repeated PropertyElement properties = 2;  // Properties to set
}
```

### Write Examples

#### Example 1: Set Simple Property

```json
// Set device name
properties: [
  {
    "name": "name",
    "value": { "v_string": "Bedroom Light" }
  }
]
```

#### Example 2: Set Multiple Properties

```json
// Set multiple properties at once
properties: [
  {
    "name": "name",
    "value": { "v_string": "Kitchen Light" }
  },
  {
    "name": "zone",
    "value": { "v_uint64": 1 }
  }
]
```

#### Example 3: Set Nested Properties

```json
// Set properties in nested structure (vDC entity)
properties: [
  {
    "name": "capabilities",
    "elements": [
      {
        "name": "metering",
        "value": { "v_bool": true }
      }
    ]
  }
]
```

### SetProperty Response

The response is a `GenericResponse`:

```protobuf
message GenericResponse {
    required ResultCode code = 1 [ default = ERR_OK ];
    optional string description = 2;
}
```

- `ERR_OK` (0): Success
- Error codes indicate failure (see [Protocol Basics](./03-protocol-basics.md))

## Property Notifications

### Push Notifications

When properties change on the vDC host side, notify the vdSM using `vdc_SendPushNotification`:

```protobuf
message vdc_SendPushNotification {
    optional string dSUID = 1;
    repeated PropertyElement changedproperties = 2;
    repeated PropertyElement deviceevents = 3;  // API v2c+
}
```

### Example: Notify State Change

```json
// Notify that a button was pressed (device entity push notification)
changedproperties: [
  {
    "name": "buttonInputStates",
    "elements": [
      {
        "name": "0",
        "elements": [
          {
            "name": "value",
            "value": { "v_bool": true }  // Button pressed
          },
          {
            "name": "clickType",
            "value": { "v_uint64": 0 }  // tip_1x
          },
          {
            "name": "age",
            "value": { "v_double": 0.0 }
          }
        ]
      }
    ]
  }
]
```

## Common Property Paths

### vDC Level Properties

When querying a vDC entity (using the vDC's dSUID):

| Path | Type | Description |
|------|------|-------------|
| `/name` | string | Human-readable vDC name |
| `/dSUID` | string | Unique identifier for the vDC |
| `/modelName` | string | Model/type of the vDC |
| `/modelVersion` | string | Version of the vDC implementation |
| `/vendorName` | string | Vendor/manufacturer name |
| `/oemModelGUID` | string | OEM model identifier |
| `/implementationId` | string | Implementation identifier |
| `/zoneID` | uint | Default zone for the vDC |
| `/capabilities` | object | vDC capabilities container |
| `/capabilities/metering` | bool | Supports power metering |
| `/capabilities/identification` | bool | Supports identify function |
| `/capabilities/dynamicDefinitions` | bool | Supports dynamic definitions |

### Device Level Properties

When querying a device (vdSD) entity (using the device's dSUID):

| Path | Type | Description |
|------|------|-------------|
| `/name` | string | Device name (user-configurable) |
| `/dSUID` | string | Unique device identifier |
| `/model` | string | Device model name |
| `/modelVersion` | string | Device model version |
| `/modelUID` | string | Model unique identifier |
| `/vendorName` | string | Device vendor |
| `/oemModelGUID` | string | OEM model GUID |
| `/hardwareVersion` | string | Hardware version |
| `/hardwareGUID` | string | Hardware GUID |
| `/serialNumber` | string | Device serial number |
| `/softwareVersion` | string | Device software version |
| `/primaryGroup` | uint | Primary functional group (color) |
| `/zoneID` | uint | Zone assignment |
| `/iconName` | string | Icon identifier for UI |

### Button Input Properties

When querying a device with button inputs:

| Path | Type | Description |
|------|------|-------------|
| `/buttonInputDescriptions` | array | Array of button input descriptors |
| `/buttonInputDescriptions/N/name` | string | Button name |
| `/buttonInputDescriptions/N/dsIndex` | uint | Button index (0..N-1) |
| `/buttonInputDescriptions/N/buttonID` | uint | Physical button ID |
| `/buttonInputDescriptions/N/buttonType` | uint | Button type (1=single, 2=2-way, etc.) |
| `/buttonInputSettings/N/group` | uint | dS group number |
| `/buttonInputSettings/N/mode` | uint | Button mode (0=standard, 2=presence, etc.) |
| `/buttonInputSettings/N/function` | uint | Button function |
| `/buttonInputStates/N/value` | bool | Current button state (pressed/released) |
| `/buttonInputStates/N/clickType` | uint | Click type (0=tip_1x, 1=tip_2x, etc.) |
| `/buttonInputStates/N/age` | double | Age of last state change (seconds) |

### Binary Input Properties

When querying a device with binary inputs:

| Path | Type | Description |
|------|------|-------------|
| `/binaryInputDescriptions` | array | Array of binary input descriptors |
| `/binaryInputDescriptions/N/name` | string | Input name |
| `/binaryInputDescriptions/N/dsIndex` | uint | Input index |
| `/binaryInputDescriptions/N/inputType` | uint | 0=poll only, 1=detects changes |
| `/binaryInputDescriptions/N/inputUsage` | uint | Input usage type |
| `/binaryInputStates/N/value` | bool | Current input state |
| `/binaryInputStates/N/age` | double | Age of last state change |

### Sensor Input Properties

When querying a device with sensors:

| Path | Type | Description |
|------|------|-------------|
| `/sensorDescriptions` | array | Array of sensor descriptors |
| `/sensorDescriptions/N/name` | string | Sensor name |
| `/sensorDescriptions/N/sensorType` | uint | Sensor type (1=temp, 2=humidity, etc.) |
| `/sensorDescriptions/N/sensorUsage` | uint | Sensor usage type |
| `/sensorDescriptions/N/min` | double | Minimum value |
| `/sensorDescriptions/N/max` | double | Maximum value |
| `/sensorStates/N/value` | double | Current sensor value |
| `/sensorStates/N/age` | double | Age of last reading |

### Output Properties

When querying a device with outputs:

| Path | Type | Description |
|------|------|-------------|
| `/outputDescription/function` | string | Output function description |
| `/outputDescription/outputUsage` | uint | Output usage type |
| `/channelDescriptions` | array | Array of channel descriptors |
| `/channelDescriptions/N/channelType` | uint | Channel type |
| `/channelDescriptions/N/channelId` | string | Channel identifier |
| `/channelDescriptions/N/name` | string | Channel name |
| `/channelDescriptions/N/min` | double | Minimum value |
| `/channelDescriptions/N/max` | double | Maximum value |
| `/channelDescriptions/N/resolution` | double | Channel resolution |
| `/channelStates/N/value` | double | Current channel value |
| `/channelStates/N/age` | double | Age of last change |

### Scene Properties

When querying a device with scene support:

| Path | Type | Description |
|------|------|-------------|
| `/scenes` | array | Scene configurations |
| `/scenes/N/sceneNo` | uint | Scene number |
| `/scenes/N/name` | string | Scene name |

## Property Types

### Value Types

PropertyValue supports these types:

| Protobuf Field | Type | Usage |
|----------------|------|-------|
| `v_bool` | boolean | Flags, capabilities, states |
| `v_uint64` | unsigned 64-bit | IDs, counters, positive numbers |
| `v_int64` | signed 64-bit | Signed numbers |
| `v_double` | double precision | Sensor values, percentages, decimals |
| `v_string` | string | Names, identifiers, text |
| `v_bytes` | binary data | Raw binary data (rare) |

### Type Selection Guidelines

- **Strings**: Use for names, IDs (dSUID), human-readable text
- **uint64**: Use for enumeration values, zone IDs, counts, non-negative integers
- **int64**: Use for signed integers
- **double**: Use for sensor values, channel values (0.0-100.0), percentages
- **bool**: Use for yes/no, enabled/disabled, true/false
- **bytes**: Rarely used, for binary data

## Implementation Best Practices

### 1. Build a Property Tree Model

Maintain separate in-memory representations for vDC and device property trees:

```python
# Example vDC property tree structure (pseudo-code)
vdc_property_tree = {
    "name": "My vDC",
    "dSUID": "abc123...",
    "modelName": "MyVdc",
    "modelVersion": "1.0",
    "vendorName": "MyCompany",
    "implementationId": "x-mycompany-myvdc",
    "zoneID": 0,
    "capabilities": {
        "metering": False,
        "identification": True,
        "dynamicDefinitions": False
    }
}

# Example device property tree structure (pseudo-code)
device_property_tree = {
    "name": "Device 1",
    "dSUID": "def456...",
    "model": "Model X",
    "modelVersion": "1.0",
    "primaryGroup": 1,
    "zoneID": 1,
    "buttonInputDescriptions": [
        {
            "name": "Button 1",
            "dsIndex": 0,
            "buttonID": 0,
            "buttonType": 1
        }
    ],
    "buttonInputSettings": [
        {
            "group": 1,
            "mode": 0,
            "function": 0
        }
    ],
    "buttonInputStates": [
        {
            "value": False,
            "clickType": 255,
            "age": 0.0
        }
    ],
    "outputDescription": {
        "function": "light",
        "outputUsage": 0
    },
    "channelDescriptions": [
        {
            "channelType": 1,
            "channelId": "brightness",
            "name": "Brightness",
            "min": 0.0,
            "max": 100.0,
            "resolution": 0.1
        }
    ],
    "channelStates": [
        {
            "value": 0.0,
            "age": 0.0
        }
    ]
}
```

### 2. Implement Property Path Resolution

Create functions to navigate your property tree:

```python
def get_property_by_path(tree, path):
    """Navigate tree using path like '/name' or '/capabilities/metering'"""
    parts = path.strip('/').split('/')
    current = tree
    for part in parts:
        if isinstance(current, list):
            current = current[int(part)]
        else:
            current = current[part]
    return current

def set_property_by_path(tree, path, value):
    """Set property value at path"""
    # Navigate to parent, then set value
    ...
```

### 3. Convert Between Formats

Implement converters between:
- Your internal representation
- PropertyElement protobuf messages

```python
def to_property_element(name, value):
    """Convert Python value to PropertyElement"""
    elem = PropertyElement()
    elem.name = name
    
    if isinstance(value, bool):
        elem.value.v_bool = value
    elif isinstance(value, str):
        elem.value.v_string = value
    elif isinstance(value, int):
        elem.value.v_uint64 = value
    elif isinstance(value, float):
        elem.value.v_double = value
    elif isinstance(value, dict):
        for key, val in value.items():
            elem.elements.append(to_property_element(key, val))
    elif isinstance(value, list):
        for idx, val in enumerate(value):
            elem.elements.append(to_property_element(str(idx), val))
    
    return elem
```

### 4. Handle Queries Efficiently

When receiving GetProperty requests:

```python
def handle_get_property(query, tree):
    """Process GetProperty query against tree"""
    result = []
    
    for query_elem in query:
        # If query has no elements, return all properties at that level
        if not query_elem.elements:
            result.append(get_all_properties(query_elem.name, tree))
        else:
            # Recursively process nested query
            result.append(get_nested_properties(query_elem, tree))
    
    return result
```

### 5. Validate Property Access

- Check if property exists before accessing
- Validate property types
- Return appropriate errors for invalid access
- Respect read-only properties

### 6. Optimize for Common Queries

Cache frequently accessed properties:
- Device name
- dSUID
- Current state values

### 7. Document Your Property Tree

Provide documentation of:
- Available properties
- Property types
- Valid ranges
- Read-only vs. read-write
- When properties change

## Advanced Topics

### Property Tree Discovery

Clients can discover your property tree by:
1. Requesting all properties: `query: [{"name": ""}]`
2. Iterating through returned structure
3. Requesting nested levels as needed

### Dynamic Properties

Some properties may be dynamic:
- Available only in certain states
- Populated based on device capabilities
- Added/removed at runtime

Handle these gracefully in GetProperty requests.

### Array Handling

Arrays are tricky in property trees:
- Elements accessed by string index: "0", "1", "2"
- May be sparse
- Consider providing array length property

### Property Metadata

Consider exposing metadata about properties:
- Min/max values
- Valid enumeration values
- Units of measurement
- Read-only flag

## Common Patterns

### Pattern 1: Initial Device Query

When vdSM connects to a device, it typically queries:

```json
query: [
  { "name": "name" },
  { "name": "dSUID" },
  { "name": "model" },
  { "name": "modelVersion" },
  { "name": "primaryGroup" },
  { "name": "outputDescription" },
  { "name": "channelDescriptions" }
]
```

Be prepared to handle this efficiently.

### Pattern 2: State Monitoring

For input devices, vdSM may periodically query button states:

```json
query: [
  {
    "name": "buttonInputStates",
    "elements": [
      { "name": "0", "elements": [{ "name": "value" }, { "name": "clickType" }] },
      { "name": "1", "elements": [{ "name": "value" }, { "name": "clickType" }] }
    ]
  }
]
```

Or for sensor values:

```json
query: [
  {
    "name": "sensorStates",
    "elements": [
      { "name": "0", "elements": [{ "name": "value" }] }
    ]
  }
]
```

Or you can push changes via PushNotification.

### Pattern 3: Configuration Update

User changes device name in UI:

```json
// SetProperty request
properties: [
  {
    "name": "name",
    "value": { "v_string": "New Name" }
  }
]
```

Your vDC host should:
1. Update internal state
2. Persist if needed
3. Return success response

## Debugging Property Trees

### Tools and Techniques

1. **Log Property Requests**: Log all GetProperty/SetProperty calls
2. **Pretty Print**: Format PropertyElement structures for readability
3. **Validate Structure**: Check that responses match query structure
4. **Test Coverage**: Test all property paths
5. **Mock Queries**: Create test queries for all expected patterns

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Empty response | Property doesn't exist | Check property name spelling |
| Wrong type | Type mismatch | Verify PropertyValue field |
| Missing nested data | Query structure wrong | Match query to tree structure |
| Array access fails | Wrong index format | Use string index "0" not int |

## Summary

The property system is the heart of vDC communication:

1. **Hierarchical Structure**: Properties organized in trees
2. **PropertyElement**: Core protobuf message for property access
3. **Query Pattern**: Request exactly what you need
4. **Typed Values**: Six value types for different data
5. **Bidirectional**: Read with GetProperty, write with SetProperty
6. **Push Updates**: Notify changes with PushNotification

**Master the property system**, and vDC implementation becomes much easier!

## Next Steps

- [vDC Reference](./05-vdc-reference.md) - vDC level properties
- [vdSD Reference](./06-vdsd-reference.md) - Device properties  
- [Device Features](./07-device-features.md) - Inputs, outputs, scenes
- [Protocol Buffer Reference](./09-protobuf-reference.md) - Complete message specs

---

[← Back: Protocol Basics](./03-protocol-basics.md) | [Back to Index](./README.md) | [Next: vDC Reference →](./05-vdc-reference.md)
