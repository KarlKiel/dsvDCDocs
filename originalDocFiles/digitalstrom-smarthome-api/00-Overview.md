# digitalSTROM Smarthome API Documentation

**Extracted from Community Implementations**

> **Note**: This documentation was reverse-engineered from open-source implementations of the digitalSTROM Smarthome API, primarily from:
> - [digitalstrom-mqtt](https://github.com/gaetancollaud/digitalstrom-mqtt) (Go implementation)
> - [homebridge-digitalSTROM](https://github.com/cgHome/homebridge-digitalSTROM) (JavaScript implementation)
> - Community forums and discussions
>
> The official API documentation from developer.digitalstrom.org/api is no longer publicly accessible. This documentation represents the best available reconstruction of the Smarthome API version 1.0.1.

## API Overview

The digitalSTROM Smarthome API is a RESTful HTTP API that provides programmatic access to digitalSTROM home automation systems. It replaced the older JSON device API with a more modern architecture.

### Base URL

```
https://{dss-host}:8080/json
```

Where `{dss-host}` is the IP address or hostname of your digitalSTROM Server (dSS).

### Authentication

All API requests require authentication using an API key (application token). The authentication flow is:

1. Obtain an API key through the digitalSTROM web interface (System → Access Authorization)
2. Login using `/system/loginApplication` with the API key to get a session token
3. Include the session token in all subsequent requests

### Request Format

All requests use HTTP GET with parameters passed in the query string:

```
GET /json/{endpoint}?param1=value1&param2=value2&token={session_token}
```

### Response Format

All responses are JSON objects with the following structure:

**Success Response:**
```json
{
  "ok": true,
  "result": {
    // Response data
  }
}
```

**Error Response:**
```json
{
  "ok": false,
  "message": "Error description"
}
```

## Key Differences from Old API

The Smarthome API introduced several significant changes:

- **Authentication**: Uses API keys instead of username/password
- **Circuits**: Replaced by "controllers" concept
- **Metering**: Power consumption moved to separate "metering" entities
- **Update Interval**: Metering data updates every 10 seconds (was 30 seconds)
- **Scenes**: Scene events are not propagated in the same way
- **Buttons**: Button events handling changed significantly
- **Device IDs**: Uses structured IDs (deviceId, outputId) instead of just dsid

## API Sections

This documentation is organized into the following sections:

1. **[Authentication](01-Authentication.md)** - Login and session management
2. **[Apartment](02-Apartment.md)** - Apartment/installation-level operations
3. **[Devices](03-Devices.md)** - Device control and status
4. **[Events](04-Events.md)** - Event subscription and notifications
5. **[Property Queries](05-Property-Queries.md)** - Advanced property tree queries
6. **[Data Structures](06-Data-Structures.md)** - Common data structures and models

## Common Concepts

### Device Identification

Devices can be identified by either:
- `dsid` - The digitalSTROM ID (unique identifier)
- `name` - The user-defined device name

### Zones and Groups

- **Zones**: Physical areas in your installation (rooms, floors)
- **Groups**: Functional groupings of devices (lights, blinds, heating)
- Special value `0` is used for broadcast operations

### Value Ranges

- Output values: 0-255 (where 255 is maximum/100%)
- Some devices support percentage values: 0-100

### Response Codes

- `HTTP 200`: Success (check `ok` field in JSON)
- `HTTP 403`: Authentication failed
- Other codes: Network or server errors
