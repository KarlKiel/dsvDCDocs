# 1. Authentication

## Overview

The digitalSTROM Smarthome API uses a two-step authentication process:
1. Exchange an API key (application token) for a session token
2. Use the session token for all subsequent API calls

## Obtaining an API Key

API keys must be obtained through the digitalSTROM web interface or using a helper script.

### Method 1: Web Interface

1. Log in to your digitalSTROM Server web interface
2. Navigate to: **System → Access Authorization**
3. Create a new API key
4. Save the key securely (it cannot be retrieved later)

### Method 2: Command-Line Tool

If you have the `digitalstrom-mqtt` tool:

```bash
./digitalstrom-mqtt -mode=get-api-key -host 192.168.1.x -username=dssadmin -password=XXX
```

This will output an API key that can be used for authentication.

## Login Endpoint

### loginApplication

Exchange an API key for a session token.

**Endpoint:** `/system/loginApplication`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| loginToken | string | Yes | The API key obtained from the web interface |

**Request Example:**

```
GET /json/system/loginApplication?loginToken=782f4e5a9b3c1d6e8f0a2b4c6d8e0f1a3b5c7d9e1f3a5b7c9d1e3f5a7b9c1d3e5
```

**Success Response:**

```json
{
  "ok": true,
  "result": {
    "token": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
  }
}
```

**Error Response:**

```json
{
  "ok": false,
  "message": "Invalid login token"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| ok | boolean | Indicates if the login was successful |
| result.token | string | Session token to be used in subsequent requests |
| message | string | Error description (only present if ok is false) |

## Using the Session Token

After successful login, include the session token in all API requests:

```
GET /json/{endpoint}?{params}&token={session_token}
```

**Example:**

```
GET /json/apartment/getDevices?token=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

## Token Management

- **Storage**: Store the session token securely in your application
- **Lifetime**: Session tokens typically remain valid for the duration of your session
- **Renewal**: If a token expires, perform the login process again
- **Security**: Never expose tokens in logs or error messages
- **Revocation**: API keys can be revoked through the web interface

## Error Handling

**HTTP 403 - Forbidden:**

If authentication fails, you will receive:
- HTTP status code: 403
- Response body:
```json
{
  "ok": false,
  "message": "not logged in"
}
```

**Action**: Perform the login process again to obtain a new session token.

## Security Considerations

1. **HTTPS**: Always use HTTPS (port 8080 by default) for API communication
2. **API Key Storage**: Store API keys securely (environment variables, secure vaults)
3. **Token Handling**: Treat session tokens as sensitive credentials
4. **Key Rotation**: Regularly rotate API keys for security
5. **Certificate Validation**: Verify SSL certificates (note: digitalSTROM often uses self-signed certificates)

## Code Examples

### JavaScript/Node.js

```javascript
const request = require('request');

const config = {
    url: 'https://192.168.1.10:8080/json',
    apiKey: '782f4e5a9b3c1d6e8f0a2b4c6d8e0f1a3b5c7d9e1f3a5b7c9d1e3f5a7b9c1d3e5'
};

// Login
request(`${config.url}/system/loginApplication?loginToken=${config.apiKey}`, 
    (error, response, body) => {
        if (!error && response.statusCode == 200) {
            const result = JSON.parse(body);
            if (result.ok) {
                const sessionToken = result.result.token;
                console.log('Login successful, token:', sessionToken);
                // Use sessionToken for subsequent requests
            }
        }
    }
);
```

### Python

```python
import requests

dss_host = "192.168.1.10"
api_key = "782f4e5a9b3c1d6e8f0a2b4c6d8e0f1a3b5c7d9e1f3a5b7c9d1e3f5a7b9c1d3e5"

# Login
response = requests.get(
    f"https://{dss_host}:8080/json/system/loginApplication",
    params={"loginToken": api_key},
    verify=False  # Note: disable certificate validation for self-signed certs
)

if response.status_code == 200:
    result = response.json()
    if result["ok"]:
        session_token = result["result"]["token"]
        print(f"Login successful, token: {session_token}")
```

### Go

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

type LoginResponse struct {
    Ok     bool `json:"ok"`
    Result struct {
        Token string `json:"token"`
    } `json:"result"`
    Message string `json:"message,omitempty"`
}

func login(host, apiKey string) (string, error) {
    url := fmt.Sprintf("https://%s:8080/json/system/loginApplication?loginToken=%s", 
                       host, apiKey)
    
    resp, err := http.Get(url)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    
    var loginResp LoginResponse
    if err := json.NewDecoder(resp.Body).Decode(&loginResp); err != nil {
        return "", err
    }
    
    if !loginResp.Ok {
        return "", fmt.Errorf("login failed: %s", loginResp.Message)
    }
    
    return loginResp.Result.Token, nil
}
```
