# 6. Data Structures

## Overview

This document describes the common data structures and objects used throughout the digitalSTROM Smarthome API, as extracted from implementation code.

## Core Entities

### Apartment

The top-level entity representing the entire installation.

**Structure:**

```json
{
  "id": "apartment-id",
  "attributes": {
    "name": "My Home",
    "zones": ["zone-id-1", "zone-id-2"],
    "dsDevices": ["device-id-1", "device-id-2"],
    "clusters": ["cluster-id-1"]
  },
  "included": {
    "installation": {},
    "devices": [],
    "submodules": [],
    "functionBlocks": [],
    "zones": [],
    "controllers": [],
    "meterings": []
  }
}
```

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| id | string | Apartment identifier |
| attributes.name | string | Installation name |
| attributes.zones | array | Array of zone IDs |
| attributes.dsDevices | array | Array of device IDs |
| attributes.clusters | array | Array of cluster IDs |
| included | object | Related entities (when using include parameter) |

### Installation

Metadata about the physical installation.

**Structure:**

```json
{
  "id": "installation-id",
  "type": "installation",
  "attributes": {
    "countryCode": "CH",
    "city": "Zurich",
    "timezone": "Europe/Zurich"
  }
}
```

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| countryCode | string | ISO country code |
| city | string | City name |
| timezone | string | IANA timezone identifier |

## Devices

### Device

Represents a physical or virtual device.

**Structure:**

```json
{
  "id": "device-id",
  "attributes": {
    "name": "Living Room Light",
    "dsid": "3504175fe000000000017ef3",
    "displayId": "GE-KL220",
    "present": true,
    "submodules": ["submodule-id-1"],
    "zone": "zone-id",
    "scenarios": ["scenario-id-1"],
    "controller": "controller-id"
  }
}
```

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| id | string | Device identifier |
| attributes.name | string | User-defined device name |
| attributes.dsid | string | digitalSTROM ID (dSUID) |
| attributes.displayId | string | Hardware display identifier |
| attributes.present | boolean | Device online/reachable status |
| attributes.submodules | array | Array of submodule IDs |
| attributes.zone | string | Zone ID where device is located |
| attributes.scenarios | array | Available scenarios for device |
| attributes.controller | string | Controller ID |

**Legacy Format (from older getDevices):**

```json
{
  "dSUID": "3504175fe000000000017ef3",
  "name": "Living Room Light",
  "groups": [1, 2],
  "zone": "32027",
  "present": true,
  "displayId": "GE-KL220"
}
```

### Submodule

A functional component of a device.

**Structure:**

```json
{
  "id": "submodule-id",
  "attributes": {
    "name": "Main Output",
    "technicalName": "output_1",
    "dsDevice": "device-id",
    "functionBlocks": ["fb-id-1"],
    "zone": "zone-id",
    "application": {
      "type": "lighting",
      "category": "indoor"
    },
    "scenarios": ["scenario-id-1"],
    "controller": "controller-id"
  }
}
```

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | User-facing name |
| technicalName | string | Technical identifier |
| dsDevice | string | Parent device ID |
| functionBlocks | array | Associated function block IDs |
| application | object | Application type and category |

### Function Block

Defines device behavior and capabilities.

**Structure:**

```json
{
  "id": "fb-id",
  "attributes": {
    "name": "Light Control",
    "technicalName": "light_ctrl_1",
    "active": true,
    "outputs": [
      {
        "id": "output-id",
        "attributes": {
          "technicalName": "brightness",
          "type": "light",
          "function": "dimming",
          "mode": "switched",
          "min": 0,
          "max": 255,
          "resolution": 1
        }
      }
    ],
    "buttonInputs": [],
    "sensorInputs": [],
    "submodule": "submodule-id",
    "deviceAdapter": "adapter-id"
  }
}
```

**Output Types:**

| Type | Description | Common Functions |
|------|-------------|------------------|
| light | Lighting output | dimming, switching, color |
| shade | Blind/shade control | position, angle |
| heating | Climate control | temperature, valve |
| audio | Audio output | volume, source |

**Output Modes:**

| Mode | Description |
|------|-------------|
| switched | On/off only |
| dimmed | Variable output (0-255) |
| color | Color control (RGB/HSV) |
| position | Positional (blinds) |

## Zones and Groups

### Zone

A physical area or room.

**Structure:**

```json
{
  "ZoneID": 32027,
  "name": "Living Room",
  "dSUID": "3504175fe000000000000001",
  "dSID": "3504175fe000000000000001",
  "isValid": true,
  "hardwareName": "dSM-Living",
  "groups": [
    {
      "group": 1,
      "name": "Light",
      "color": 1,
      "applicationType": "Lighting",
      "scenes": [],
      "devices": []
    }
  ]
}
```

**Standard Zones:**

| ZoneID | Name | Description |
|--------|------|-------------|
| 0 | Broadcast | All zones |
| 32xxx | User-defined | Custom zones/rooms |
| 65535 | Not assigned | Devices not in a zone |

### Group

A functional grouping within a zone.

**Structure:**

```json
{
  "group": 1,
  "name": "Light",
  "color": 1,
  "applicationType": "Lighting",
  "scenes": {
    "5": {
      "scene": 5,
      "name": "Preset 1"
    }
  },
  "devices": ["device-dsuid-1", "device-dsuid-2"]
}
```

**Standard Groups:**

| ID | Color | Name | Application Type |
|----|-------|------|------------------|
| 0 | - | Broadcast | All groups |
| 1 | Yellow | Light | Lighting |
| 2 | Gray | Blinds | Blinds/Shades |
| 3 | Blue | Climate | Heating/Cooling |
| 4 | Cyan | Audio | Audio/Media |
| 5 | Magenta | Video | Video |
| 8 | Red | Security | Security/Access |
| 48 | Black | Joker | Multi-function |

## Controllers and Metering

### Controller

A digitalSTROM controller/meter (replaces old "Circuit" concept).

**Structure:**

```json
{
  "id": "controller-id",
  "attributes": {
    "name": "dSM12 Main",
    "dsid": "3504175fe000000000001234",
    "hardwareType": "dSM12",
    "present": true,
    "zones": ["zone-id-1", "zone-id-2"]
  }
}
```

### Metering

Power consumption data entity.

**Structure:**

```json
{
  "id": "metering-id",
  "attributes": {
    "name": "Main Metering",
    "controller": "controller-id",
    "powerW": 1250.5,
    "energyWh": 150000,
    "timestamp": "2024-01-09T17:00:00Z"
  }
}
```

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| powerW | number | Current power consumption in Watts |
| energyWh | number | Total energy consumption in Watt-hours |
| timestamp | string | ISO 8601 timestamp |

**Update Interval:** Metering data updates every 10 seconds (changed from 30 seconds in old API)

**Metering Hierarchy:**

- **Apartment Metering**: Sum of all controller meterings
- **Controller Metering**: Individual controller measurements

## Events

### Event Object

**Structure:**

```json
{
  "name": "callScene",
  "source": {
    "dsid": "3504175fe000000000017ef3",
    "dSUID": "3504175fe000000000017ef3",
    "zoneID": 32027,
    "groupID": 1
  },
  "properties": {
    "sceneID": 5,
    "sceneId": 5,
    "forced": false,
    "groupID": 1,
    "zoneID": 32027
  }
}
```

**Event Types:**

| Name | Description |
|------|-------------|
| callScene | Scene activation |
| scene_name_changed | Scene renamed |
| device_sensor_value | Sensor value change |
| device_status | Device status change |

## Scenarios/Scenes

### Scene

**Structure:**

```json
{
  "scene": 5,
  "name": "Preset 1",
  "dontCare": false,
  "localPriority": false,
  "ignoreLocalPriority": false
}
```

**Common Scene IDs:**

| Scene ID | Name | Type | Description |
|----------|------|------|-------------|
| 0 | Off | System | Turn off |
| 1-4 | Area 1-4 | System | Predefined areas |
| 5, 17-19 | Preset 1-4 | User | User-defined |
| 14 | On | System | Turn on |
| 15 | Increment | System | Increase |
| 16 | Decrement | System | Decrease |
| 13 | Stop | System | Stop movement (blinds) |
| 64-68 | Auto Off | Auto | Auto-off timers |

## Value Ranges and Units

### Common Value Ranges

| Type | Min | Max | Unit | Description |
|------|-----|-----|------|-------------|
| Output Value | 0 | 255 | - | Device output (100% = 255) |
| Brightness | 0 | 100 | % | Light brightness |
| Position | 0 | 100 | % | Blind position (0=open, 100=closed)* |
| Angle | 0 | 90 | degrees | Blind slat angle |
| Temperature | -273 | +inf | °C | Temperature value |
| Power | 0 | +inf | W | Power consumption |
| Energy | 0 | +inf | Wh | Energy consumption |

*Note: Some implementations invert blind position (0=closed, 100=open)

### Conversion Examples

**Percentage to 0-255:**
```javascript
value255 = Math.round(percentage / 100 * 255);
```

**0-255 to Percentage:**
```javascript
percentage = Math.round(value255 / 255 * 100);
```

## Response Structures

### Success Response

```json
{
  "ok": true,
  "result": {
    // Response data
  }
}
```

### Error Response

```json
{
  "ok": false,
  "message": "Error description"
}
```

### Common Error Messages

| Message | Cause | HTTP Code |
|---------|-------|-----------|
| "not logged in" | Invalid or expired session token | 403 |
| "Unhandled function" | Invalid endpoint | 404 |
| "device not found" | Invalid device ID | 404 |
| "device not present" | Device offline | 400 |
| "invalid value" | Value out of range | 400 |

## Type Definitions (TypeScript)

```typescript
interface Apartment {
  id: string;
  attributes: {
    name: string;
    zones: string[];
    dsDevices: string[];
    clusters: string[];
  };
  included?: ApartmentIncluded;
}

interface Device {
  id: string;
  attributes: {
    name: string;
    dsid: string;
    displayId: string;
    present: boolean;
    submodules: string[];
    zone: string;
    scenarios: string[];
    controller: string;
  };
}

interface Zone {
  ZoneID: number;
  name: string;
  dSUID: string;
  isValid: boolean;
  groups: Group[];
}

interface Group {
  group: number;
  name: string;
  color: number;
  applicationType: string;
  scenes: { [key: number]: Scene };
  devices: string[];
}

interface Scene {
  scene: number;
  name: string;
  dontCare?: boolean;
}

interface Metering {
  id: string;
  attributes: {
    name: string;
    controller: string;
    powerW: number;
    energyWh: number;
    timestamp: string;
  };
}
```

## Notes

- Data structures may vary slightly between API versions
- Optional fields may not be present in all responses
- Some legacy fields (dSID vs dSUID) exist for backward compatibility
- Always check `present` field before device operations
- Metering data is updated every 10 seconds
- Controllers replace the old "Circuit" concept
