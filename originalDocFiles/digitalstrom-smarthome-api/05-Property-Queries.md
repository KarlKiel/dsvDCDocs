# 5. Property Queries

## Overview

The Property Query API provides a powerful way to query the digitalSTROM property tree using path-based queries. This is an advanced feature that allows flexible access to system data.

## Property Tree Structure

The digitalSTROM system organizes data in a hierarchical tree structure:

```
/apartment
  /dSMeters/*
    /zones/*
      (properties...)
      /groups/*
        (properties...)
        /scenes/*
          (properties...)
        /devices/*
          (properties...)
```

## Query Endpoint

### property/query

Execute a property tree query.

**Endpoint:** `/property/query`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| query | string | Yes | Property path query string |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/property/query?query=/apartment/*/*(ZoneID,name)/groups/*(group)&token=...
```

## Query Syntax

### Path Elements

| Element | Description | Example |
|---------|-------------|---------|
| `/apartment` | Root path | `/apartment` |
| `/dSMeters` | Access dSMeters | `/apartment/dSMeters` |
| `/*` | Wildcard (all items) | `/apartment/*/zones/*` |
| `/{id}` | Specific item by ID | `/apartment/dSMeters/3504175fe000` |
| `(prop1,prop2)` | Select properties | `(ZoneID,name,dSUID)` |
| `(*)` | All properties | `/scenes/*(*)` |

### Basic Examples

**Get all zone IDs and names:**
```
/apartment/*/*(ZoneID,name)
```

**Get all groups in all zones:**
```
/apartment/*/zones/*/groups/*
```

**Get specific zone information:**
```
/apartment/dSMeters/*/zones/32027(ZoneID,name,dSUID)
```

## Common Query Patterns

### 1. Get Zones with Basic Info

**Query:**
```
/apartment/*/*(ZoneID,name,dSUID,isValid)
```

**Response:**
```json
{
  "ok": true,
  "result": {
    "zones": [
      {
        "ZoneID": 32027,
        "name": "Living Room",
        "dSUID": "3504175fe000000000000001",
        "isValid": true
      }
      // More zones...
    ]
  }
}
```

### 2. Get All Groups with Scenes

**Query:**
```
/apartment/*/*(ZoneID,name)/groups/*(group,name,color,applicationType)/scenes/*(*)
```

**Response:**
```json
{
  "ok": true,
  "result": {
    "dSMeters": [
      {
        "zones": [
          {
            "ZoneID": 32027,
            "name": "Living Room",
            "groups": [
              {
                "group": 1,
                "name": "Light",
                "color": 1,
                "applicationType": "Lighting",
                "scenes": [
                  {
                    "scene": 5,
                    "name": "Preset 1",
                    // All scene properties...
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

### 3. Get Devices in Groups

**Query:**
```
/apartment/*/*(ZoneID,name)/groups/*(group,name)/devices/*(dSUID)
```

**Response:**
```json
{
  "ok": true,
  "result": {
    "dSMeters": [
      {
        "zones": [
          {
            "ZoneID": 32027,
            "name": "Living Room",
            "groups": [
              {
                "group": 1,
                "name": "Light",
                "devices": [
                  {
                    "dSUID": "3504175fe000000000017ef3"
                  },
                  {
                    "dSUID": "3504175fe000000000018a45"
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

### 4. Get Complete Scene Information

**Query:**
```
/apartment/*/*(ZoneID,name,dSUID,dSID,isValid,hardwareName)/groups/*(group,name,color,applicationType)/scenes/*(*)
```

This returns comprehensive scene data including:
- Zone information
- Group details
- All scene properties (name, ID, configuration, etc.)

### 5. Get Hardware and Device Details

**Query:**
```
/apartment/dSMeters/*/zones/*(ZoneID,name,hardwareName,dSID)/groups/*(group)/devices/*(dSUID,name,present)
```

Returns detailed hardware and device information.

## Property Selectors

### Common Zone Properties

| Property | Type | Description |
|----------|------|-------------|
| ZoneID | number | Unique zone identifier |
| name | string | Zone name |
| dSUID | string | Zone dSUID |
| dSID | string | Zone dSID |
| isValid | boolean | Zone validity status |
| hardwareName | string | Hardware identifier |

### Common Group Properties

| Property | Type | Description |
|----------|------|-------------|
| group | number | Group ID (1-8, 48) |
| name | string | Group name |
| color | number | Color code |
| applicationType | string | Application type |

### Common Scene Properties

| Property | Type | Description |
|----------|------|-------------|
| scene | number | Scene ID |
| name | string | Scene name |
| dontCare | boolean | Don't care flag |

### Common Device Properties

| Property | Type | Description |
|----------|------|-------------|
| dSUID | string | Device dSUID |
| name | string | Device name |
| present | boolean | Device presence |
| dsid | string | Device dSID |

## Advanced Usage

### Combining Multiple Levels

Query multiple levels of the hierarchy in one request:

```
/apartment/dSMeters/3504175fe000/zones/*/groups/1/devices/*(dSUID,name,present)
```

This gets all light devices (group 1) across all zones for a specific dSMeter.

### Using Include Parameter

Some endpoints support an `include` parameter to fetch related data:

```
/apartment/getDetails?include=zones,devices,controllers&token=...
```

## Performance Considerations

1. **Specificity**: More specific queries are faster
2. **Property Selection**: Only request properties you need
3. **Wildcards**: Each wildcard increases processing time
4. **Caching**: Consider caching query results for static data
5. **Pagination**: Large result sets may need pagination handling

## Code Examples

### JavaScript: Get All Lights in All Zones

```javascript
async function getAllLights(token) {
    const query = encodeURIComponent(
        '/apartment/*/*(ZoneID,name)/groups/1(group,name)/devices/*(dSUID,name,present)'
    );
    
    const response = await fetch(
        `https://dss.local:8080/json/property/query?query=${query}&token=${token}`
    );
    
    const data = await response.json();
    
    // Parse nested structure
    const lights = [];
    data.result.dSMeters.forEach(meter => {
        meter.zones.forEach(zone => {
            zone.groups.forEach(group => {
                if (group.devices) {
                    group.devices.forEach(device => {
                        lights.push({
                            ...device,
                            zoneName: zone.name,
                            zoneID: zone.ZoneID
                        });
                    });
                }
            });
        });
    });
    
    return lights;
}
```

### Python: Get Scene Configuration

```python
def get_scenes(token, dss_host="192.168.1.10"):
    query = '/apartment/*/*(ZoneID,name)/groups/*(group,name)/scenes/*(scene,name)'
    
    resp = requests.get(
        f"https://{dss_host}:8080/json/property/query",
        params={"query": query, "token": token},
        verify=False
    )
    
    data = resp.json()["result"]
    
    # Flatten nested structure
    scenes = []
    for meter in data["dSMeters"]:
        for zone in meter["zones"]:
            for group in zone["groups"]:
                if "scenes" in group:
                    for scene in group["scenes"]:
                        scenes.append({
                            **scene,
                            "zoneName": zone["name"],
                            "zoneID": zone["ZoneID"],
                            "groupName": group["name"],
                            "groupID": group["group"]
                        })
    
    return scenes
```

## Best Practices

1. **URL Encoding**: Always URL-encode query strings
2. **Error Handling**: Handle malformed query syntax errors
3. **Result Parsing**: Be prepared for nested data structures
4. **Null Checks**: Some properties may not exist for all items
5. **Query Testing**: Test queries with simple paths first
6. **Documentation**: Document complex queries in your code

## Troubleshooting

**Empty Results:**
- Check query syntax
- Verify path exists (some zones/groups may be empty)
- Confirm wildcards are correct

**Syntax Errors:**
- Ensure parentheses are balanced
- Check for typos in property names
- Verify path separators (/)

**Performance Issues:**
- Reduce number of wildcards
- Limit property selection
- Query specific paths instead of broad wildcards

## Notes

- Property queries are powerful but complex
- For simple needs, use dedicated endpoints (getDevices, getStructure)
- Query results follow the tree structure closely
- Some properties may be read-only via queries
- Not all system data is accessible via property queries
- Query syntax may vary slightly between API versions
