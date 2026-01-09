# 3. Device Operations

## Overview

Device operations allow you to control individual devices, read their status, and configure their settings. Devices are identified by their `dsid` (digitalSTROM ID) or `name`.

## Device Control Endpoints

### turnOn

Turn on a device.

**Endpoint:** `/device/turnOn`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

*Either `dsid` or `name` must be provided

**Request Example:**

```
GET /json/device/turnOn?dsid=3504175fe000000000017ef3&token=...
```

**Success Response:**

```json
{
  "ok": true
}
```

### turnOff

Turn off a device.

**Endpoint:** `/device/turnOff`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/device/turnOff?dsid=3504175fe000000000017ef3&token=...
```

### setValue

Set the output value of a device.

**Endpoint:** `/device/setValue`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| value | number | Yes | Output value (0-255) |
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/device/setValue?value=128&dsid=3504175fe000000000017ef3&token=...
```

**Value Range:**

- `0`: Off/Minimum
- `128`: 50% brightness/position
- `255`: Full/Maximum

**Usage Notes:**

- For lights: Value represents brightness (0=off, 255=full brightness)
- For blinds: Value represents position (0=fully open, 255=fully closed)*
  - *Note: Some implementations invert this (100% = fully closed)
- For dimmers: Value represents output percentage

## Device Status Endpoints

### getOutputValue

Get the current output value of a device.

**Endpoint:** `/device/getOutputValue`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| offset | number | No | Output offset (default: 0) |
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/device/getOutputValue?offset=0&dsid=3504175fe000000000017ef3&token=...
```

**Success Response:**

```json
{
  "ok": true,
  "result": {
    "value": 128
  }
}
```

### getOutputChannelValue

Get specific output channel values (for multi-channel devices).

**Endpoint:** `/device/getOutputChannelValue`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| channels | string | Yes | Channel name (e.g., "shadeOpeningAngleIndoor") |
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/device/getOutputChannelValue?channels=shadeOpeningAngleIndoor&dsid=...&token=...
```

**Common Channel Names:**

| Channel | Device Type | Description |
|---------|-------------|-------------|
| shadeOpeningAngleIndoor | Blinds | Indoor shade angle |
| shadeOpeningAngleOutside | Blinds | Outdoor shade angle |
| shadePositionIndoor | Blinds | Indoor shade position |
| shadePositionOutside | Blinds | Outdoor shade position |

**Success Response:**

```json
{
  "ok": true,
  "result": {
    "shadeOpeningAngleIndoor": 45
  }
}
```

## Device Configuration

### getConfig

Get a device configuration value.

**Endpoint:** `/device/getConfig`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| class | number | Yes | Configuration class |
| index | number | Yes | Configuration parameter index |
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/device/getConfig?class=64&index=4&dsid=...&token=...
```

**Success Response:**

```json
{
  "ok": true,
  "result": {
    "value": 200
  }
}
```

### setConfig

Set a device configuration value.

**Endpoint:** `/device/setConfig`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| class | number | Yes | Configuration class |
| index | number | Yes | Configuration parameter index |
| value | number | Yes | Configuration value to set |
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/device/setConfig?class=64&index=4&value=200&dsid=...&token=...
```

**Common Configuration Parameters:**

| Class | Index | Description | Example Value |
|-------|-------|-------------|---------------|
| 64 | 4 | Shade tilt angle | 0-255 |
| 64 | 5 | Shade position offset | 0-255 |

*Note: Configuration parameters are device-specific. Consult device documentation.*

## Device Identification

### blink

Make a device blink for identification purposes.

**Endpoint:** `/device/blink`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| dsid | string | Yes* | Device dSUID |
| name | string | Yes* | Device name |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/device/blink?dsid=3504175fe000000000017ef3&token=...
```

**Success Response:**

```json
{
  "ok": true
}
```

**Behavior:**

- Lights: Blink on/off several times
- Other devices: May flash an indicator LED or perform a brief movement

## Code Examples

### JavaScript: Dim a Light to 50%

```javascript
async function dimLight(dsid, sessionToken) {
    const value = Math.round(0.5 * 255); // 50% = 128
    
    const response = await fetch(
        `https://dss.local:8080/json/device/setValue?value=${value}&dsid=${dsid}&token=${sessionToken}`
    );
    
    const result = await response.json();
    return result.ok;
}
```

### Python: Toggle Device State

```python
import requests

def toggle_device(dsid, token, dss_host="192.168.1.10"):
    # Get current value
    resp = requests.get(
        f"https://{dss_host}:8080/json/device/getOutputValue",
        params={"offset": 0, "dsid": dsid, "token": token},
        verify=False
    )
    
    current_value = resp.json()["result"]["value"]
    
    # Toggle: if on (>0), turn off; if off, turn on
    action = "turnOff" if current_value > 0 else "turnOn"
    
    resp = requests.get(
        f"https://{dss_host}:8080/json/device/{action}",
        params={"dsid": dsid, "token": token},
        verify=False
    )
    
    return resp.json()["ok"]
```

### Go: Set Blind Position

```go
func setBlindPosition(dsid, token string, percentage int) error {
    // Convert percentage to 0-255 range
    value := int(float64(percentage) / 100.0 * 255)
    
    url := fmt.Sprintf(
        "https://dss.local:8080/json/device/setValue?value=%d&dsid=%s&token=%s",
        value, dsid, token,
    )
    
    resp, err := http.Get(url)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
    
    if !result["ok"].(bool) {
        return fmt.Errorf("failed to set blind position")
    }
    
    return nil
}
```

## Best Practices

1. **Use dSUID over name**: dSUID is guaranteed unique, names may change
2. **Check response**: Always verify the `ok` field in responses
3. **Handle presence**: Check device `present` status before operations
4. **Throttle requests**: Don't spam the API with rapid requests
5. **Value conversion**: Remember 0-255 range, convert from percentages appropriately
6. **Multi-channel devices**: Use `getOutputChannelValue` for complex devices like blinds

## Error Handling

**Common Errors:**

| Error | Cause | Solution |
|-------|-------|----------|
| "device not found" | Invalid dsid or name | Verify device exists using `/apartment/getDevices` |
| "device not present" | Device offline | Check device connectivity |
| "invalid value" | Value out of range | Use 0-255 for setValue |
| "not authorized" | Permission issue | Check device permissions |

## Notes

- Some operations may take time to complete (e.g., blinds moving)
- Devices may have hardware limits that prevent certain values
- Configuration parameters vary by device model and manufacturer
- Always test blink on a non-critical device first
