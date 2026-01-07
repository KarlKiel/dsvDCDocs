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

```
/ (root)
├── vdc                      [container]
│   ├── name                 [string value]
│   ├── dSUID                [string value]
│   ├── modelName            [string value]
│   ├── modelVersion         [string value]
│   └── capabilities         [container]
│       ├── metering         [bool value]
│       └── identification   [bool value]
│
└── device                   [container]
    ├── name                 [string value]
    ├── dSUID                [string value]
    ├── model                [string value]
    ├── modelVersion         [string value]
    ├── inputs               [array/container]
    │   ├── [0]              [container]
    │   │   ├── inputType    [number value]
    │   │   ├── name         [string value]
    │   │   └── state        [number value]
    │   └── [1]              [container]
    │       └── ...
    ├── outputs              [container]
    │   └── ...
    └── scenes               [container]
        └── ...
```

### Property Paths

Properties are accessed using **path notation**:

- Root: `/`
- Simple path: `/vdc/name`
- Nested path: `/device/outputs/channels/0/name`
- Array element: `/device/inputs/2/state`

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
// Query for device model information
query: [
  {
    "name": "device",
    "elements": [
      { "name": "model" },
      { "name": "modelVersion" }
    ]
  }
]

// Response:
properties: [
  {
    "name": "device",
    "elements": [
      {
        "name": "model",
        "value": { "v_string": "Hue Color Bulb" }
      },
      {
        "name": "modelVersion",
        "value": { "v_string": "1.0" }
      }
    ]
  }
]
```

#### Query Pattern 3: Request All Properties

Omit the `elements` field to request all child properties:

```json
// Query for all vdc properties
query: [
  {
    "name": "vdc"
    // No elements field = get everything under vdc
  }
]

// Response: All vdc properties returned
properties: [
  {
    "name": "vdc",
    "elements": [
      { "name": "name", "value": { "v_string": "Philips Hue vDC" } },
      { "name": "dSUID", "value": { "v_string": "..." } },
      { "name": "modelName", "value": { "v_string": "..." } },
      // ... all other vdc properties
    ]
  }
]
```

#### Query Pattern 4: Array Access

Access array elements by index:

```json
// Query for first input
query: [
  {
    "name": "inputs",
    "elements": [
      {
        "name": "0",  // Array index as string
        "elements": [
          { "name": "inputType" },
          { "name": "name" }
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
// Set properties in nested structure
properties: [
  {
    "name": "device",
    "elements": [
      {
        "name": "name",
        "value": { "v_string": "New Device Name" }
      },
      {
        "name": "zone",
        "value": { "v_uint64": 2 }
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
// Notify that a button was pressed
changedproperties: [
  {
    "name": "inputs",
    "elements": [
      {
        "name": "0",
        "elements": [
          {
            "name": "state",
            "value": { "v_uint64": 1 }  // Button pressed
          }
        ]
      }
    ]
  }
]
```

## Common Property Paths

### vDC Level Properties

| Path | Type | Description |
|------|------|-------------|
| `/vdc/name` | string | Human-readable vDC name |
| `/vdc/dSUID` | string | Unique identifier for the vDC |
| `/vdc/modelName` | string | Model/type of the vDC |
| `/vdc/modelVersion` | string | Version of the vDC implementation |
| `/vdc/vendorName` | string | Vendor/manufacturer name |
| `/vdc/oemModelGUID` | string | OEM model identifier |
| `/vdc/capabilities` | object | vDC capabilities container |
| `/vdc/capabilities/metering` | bool | Supports power metering |
| `/vdc/capabilities/identification` | bool | Supports identify function |

### Device Level Properties

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

### Input Properties

| Path | Type | Description |
|------|------|-------------|
| `/inputs` | array | Array of input descriptors |
| `/inputs/N/inputType` | uint | Input type (button, binary, sensor) |
| `/inputs/N/name` | string | Input name |
| `/inputs/N/state` | uint/double | Current input state |
| `/inputs/N/age` | uint | Age of last state change (seconds) |

### Output Properties

| Path | Type | Description |
|------|------|-------------|
| `/output` | object | Output configuration container |
| `/output/channels` | array | Output channels |
| `/output/channels/N/channelType` | uint | Channel type |
| `/output/channels/N/name` | string | Channel name |
| `/output/channels/N/resolution` | double | Channel resolution |
| `/output/channels/N/min` | double | Minimum value |
| `/output/channels/N/max` | double | Maximum value |

### Scene Properties

| Path | Type | Description |
|------|------|-------------|
| `/scenes` | array | Scene configurations |
| `/scenes/N/sceneNo` | uint | Scene number |
| `/scenes/N/name` | string | Scene name |
| `/scenes/N/channels` | array | Channel values for scene |

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

Maintain an in-memory representation of your property tree:

```python
# Example property tree structure (pseudo-code)
property_tree = {
    "vdc": {
        "name": "My vDC",
        "dSUID": "abc123...",
        "capabilities": {
            "metering": False,
            "identification": True
        }
    },
    "name": "Device 1",
    "dSUID": "def456...",
    "model": "Model X",
    "inputs": [
        {
            "inputType": 1,
            "name": "Button 1",
            "state": 0
        }
    ]
}
```

### 2. Implement Property Path Resolution

Create functions to navigate your property tree:

```python
def get_property_by_path(tree, path):
    """Navigate tree using path like '/vdc/name'"""
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
  { "name": "output" }
]
```

Be prepared to handle this efficiently.

### Pattern 2: State Monitoring

For input devices, vdSM may periodically query:

```json
query: [
  {
    "name": "inputs",
    "elements": [
      { "name": "0", "elements": [{ "name": "state" }] },
      { "name": "1", "elements": [{ "name": "state" }] }
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
