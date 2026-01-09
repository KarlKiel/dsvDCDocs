# 2. Apartment Operations

## Overview

The Apartment API provides access to apartment-level (installation-level) operations, including retrieving device lists, structure information, and overall system status.

## Endpoints

### getDevices

Get a list of all devices in the apartment/installation.

**Endpoint:** `/apartment/getDevices`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| token | string | Yes | Session token from login |

**Request Example:**

```
GET /json/apartment/getDevices?token=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

**Success Response:**

```json
{
  "ok": true,
  "result": [
    {
      "dSUID": "3504175fe000000000017ef3",
      "name": "Living Room Light",
      "groups": [1, 2],
      "zone": "32027",
      "present": true,
      "displayId": "GE-KL220",
      // Additional device properties...
    },
    {
      "dSUID": "3504175fe000000000018a45",
      "name": "Kitchen Blind",
      "groups": [2, 4],
      "zone": "32028",
      "present": true,
      //...
    }
    // More devices...
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| dSUID | string | Unique device identifier |
| name | string | User-defined device name |
| groups | array | Array of group IDs the device belongs to |
| zone | string | Zone ID where the device is located |
| present | boolean | Whether the device is currently reachable/online |
| displayId | string | Hardware display identifier |

**Usage Notes:**

- This endpoint returns all devices regardless of their zone or group
- Devices may belong to multiple groups simultaneously
- The `present` field indicates if the device is currently responsive

### getStructure

Get the structural organization of the apartment including zones and groups.

**Endpoint:** `/apartment/getStructure`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| token | string | Yes | Session token from login |

**Request Example:**

```
GET /json/apartment/getStructure?token=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

**Success Response:**

```json
{
  "ok": true,
  "result": {
    "zones": [
      {
        "id": "32027",
        "name": "Living Room",
        "groups": [
          {
            "id": 1,
            "name": "Light",
            "color": 1,
            "applicationType": "Lighting"
          },
          {
            "id": 2,
            "name": "Blinds",
            "color": 2,
            "applicationType": "Blinds"
          }
        ]
      },
      {
        "id": "32028",
        "name": "Kitchen",
        "groups": [...]
      }
    ],
    "dSMeters": [...],
    "clusters": [...]
  }
}
```

**Response Structure:**

| Field | Type | Description |
|-------|------|-------------|
| zones | array | Array of zone objects with their groups |
| dSMeters | array | Array of digitalSTROM meter information |
| clusters | array | Array of cluster definitions |

**Zone Object:**

| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique zone identifier |
| name | string | User-defined zone name |
| groups | array | Groups available in this zone |

**Group Object:**

| Field | Type | Description |
|-------|------|-------------|
| id | number | Group identifier (1=Yellow/Light, 2=Gray/Blinds, 3=Blue/Climate, 4=Cyan/Audio, 8=Magenta/Security) |
| name | string | Group name |
| color | number | Color code for the group |
| applicationType | string | Type of application/function |

**Standard Group IDs:**

| Group ID | Color | Name | Application Type |
|----------|-------|------|------------------|
| 1 | Yellow | Light | Lighting |
| 2 | Gray | Blinds | Blinds/Shades |
| 3 | Blue | Climate | Heating/Cooling |
| 4 | Cyan | Audio | Audio/Media |
| 5 | Magenta | Video | Video |
| 8 | Red | Security | Security/Access |
| 48 | Black | Joker | Multi-function |

## Property Query (Advanced)

For more complex queries, use the property query endpoint.

**Endpoint:** `/property/query`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| query | string | Yes | Property path query string |
| token | string | Yes | Session token |

**Example: Get all zones with devices:**

```
GET /json/property/query?query=/apartment/*/*(ZoneID,name)/groups/*(group,name,color)/devices/*(dSUID)&token=...
```

**Example: Get scenes for all zones and groups:**

```
GET /json/property/query?query=/apartment/*/*(ZoneID,name,dSUID)/groups/*(group,name)/scenes/*(*)&token=...
```

**Property Query Syntax:**

- `/apartment`: Root path
- `/*`: Wildcard for dSMeters
- `/*`: Wildcard for zones
- `(ZoneID,name)`: Select specific properties
- `/groups/*`: All groups
- `*(group,name)`: Group properties
- `/scenes/*`: All scenes in group
- `(*)`: All scene properties

## Common Use Cases

### 1. Get All Devices and Their Locations

```javascript
// First get structure
const structure = await getStructure();

// Then get devices
const devices = await getDevices();

// Combine to create a complete map
const deviceMap = devices.map(device => ({
    ...device,
    zoneName: structure.zones.find(z => z.id === device.zone)?.name,
    groupNames: device.groups.map(g => 
        structure.zones
            .flatMap(z => z.groups)
            .find(grp => grp.id === g)?.name
    )
}));
```

### 2. Get All Lights in Living Room

```javascript
const structure = await getStructure();
const devices = await getDevices();

const livingRoom = structure.zones.find(z => z.name === "Living Room");
const lightGroup = livingRoom.groups.find(g => g.id === 1); // Yellow/Light group

const livingRoomLights = devices.filter(d => 
    d.zone === livingRoom.id && d.groups.includes(1)
);
```

### 3. Check Device Availability

```javascript
const devices = await getDevices();
const unavailableDevices = devices.filter(d => !d.present);

if (unavailableDevices.length > 0) {
    console.log("Offline devices:", unavailableDevices.map(d => d.name));
}
```

## Error Handling

**Common Errors:**

| Error | Description | Solution |
|-------|-------------|----------|
| "not logged in" | Session token expired or invalid | Re-authenticate using `/system/loginApplication` |
| "Unhandled function" | Endpoint not found | Check endpoint spelling and API version compatibility |
| HTTP 403 | Authentication failure | Verify token is included and valid |

## Notes

- Zone ID `0` represents broadcast (all zones)
- Group ID `0` represents broadcast (all groups) within a zone
- The structure includes virtual dSMeters which may not have physical devices
- Property queries provide more flexibility but are more complex to use
- Some devices may belong to no groups (e.g., system devices)
