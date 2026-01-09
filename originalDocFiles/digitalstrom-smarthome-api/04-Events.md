# 4. Events and Notifications

## Overview

The Events API provides real-time notifications for changes in the digitalSTROM system using a subscription-based model with long-polling.

## Event Subscription Flow

1. Subscribe to specific event types using `/event/subscribe`
2. Poll for events using `/event/get` with long-polling (timeout)
3. Process received events
4. Repeat step 2 (the long-polling loop)

## Endpoints

### subscribe

Subscribe to a specific event type.

**Endpoint:** `/event/subscribe`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| subscriptionID | number | Yes | Unique identifier for this subscription (client-chosen) |
| name | string | Yes | Event name to subscribe to |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/event/subscribe?subscriptionID=42&name=callScene&token=...
```

**Success Response:**

```json
{
  "ok": true
}
```

**Available Event Names:**

| Event Name | Description |
|------------|-------------|
| callScene | Triggered when a scene is called |
| scene_name_changed | Triggered when a scene's name is modified |
| device_sensor_value | Sensor value changes |
| device_status | Device status changes |
| buttonClick | Button press events (if available) |

*Note: Availability varies by API version and system configuration*

### get

Retrieve events for a subscription (long-polling).

**Endpoint:** `/event/get`

**Method:** GET

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| subscriptionID | number | Yes | Subscription ID from subscribe call |
| timeout | number | No | Timeout in milliseconds (default: 60000) |
| token | string | Yes | Session token |

**Request Example:**

```
GET /json/event/get?subscriptionID=42&timeout=60000&token=...
```

**Success Response:**

```json
{
  "ok": true,
  "result": {
    "events": [
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
    ]
  }
}
```

**Long-Polling Behavior:**

- The request will wait up to `timeout` milliseconds for events
- If events occur, they are returned immediately
- If no events occur before timeout, an empty events array is returned
- Client should immediately make another `get` request (continuous polling)

## Event Structure

### callScene Event

Triggered when a scene is activated.

**Event Object:**

```json
{
  "name": "callScene",
  "source": {
    "dsid": "device_id",
    "dSUID": "device_id",
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

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | Event type ("callScene") |
| source.dsid | string | Device that triggered the scene (may be virtual) |
| source.dSUID | string | Same as dsid |
| source.zoneID | number | Zone where scene was called |
| source.groupID | number | Group affected by scene |
| properties.sceneID | number | Scene number (0-127) |
| properties.forced | boolean | Whether scene was force-called |

**Common Scene IDs:**

| Scene ID | Name | Description |
|----------|------|-------------|
| 5 | Preset 1 | First saved scene |
| 17 | Preset 2 | Second saved scene |
| 18 | Preset 3 | Third saved scene |
| 19 | Preset 4 | Fourth saved scene |
| 0 | Off | Turn off |
| 14 | On | Turn on |
| 15 | Increment | Increase value |
| 16 | Decrement | Decrease value |

### scene_name_changed Event

Triggered when a scene's name is modified.

**Event Object:**

```json
{
  "name": "scene_name_changed",
  "source": {
    "zoneID": 32027,
    "groupID": 1,
    "sceneID": 5
  },
  "properties": {
    "sceneName": "Evening Light",
    "sceneID": 5
  }
}
```

## Implementation Pattern

### JavaScript/Node.js

```javascript
class DSEventListener {
    constructor(token, eventName) {
        this.token = token;
        this.eventName = eventName;
        this.subscriptionID = Math.floor(Math.random() * 1000);
    }
    
    async start() {
        // Subscribe
        await this.request(`/event/subscribe?subscriptionID=${this.subscriptionID}&name=${this.eventName}`);
        
        // Start long-polling loop
        this.longPoll();
    }
    
    async longPoll() {
        try {
            const result = await this.request(
                `/event/get?subscriptionID=${this.subscriptionID}&timeout=60000`
            );
            
            // Process events
            if (result.events && result.events.length > 0) {
                result.events.forEach(event => this.handleEvent(event));
            }
            
            // Continue polling
            setImmediate(() => this.longPoll());
            
        } catch (error) {
            console.error('Long poll error:', error);
            setTimeout(() => this.longPoll(), 5000); // Retry after 5s
        }
    }
    
    handleEvent(event) {
        console.log('Event received:', event.name);
        
        if (event.name === 'callScene') {
            console.log(`Scene ${event.properties.sceneID} called in zone ${event.source.zoneID}`);
        }
    }
    
    async request(path) {
        const response = await fetch(
            `https://dss.local:8080/json${path}&token=${this.token}`
        );
        const data = await response.json();
        return data.result;
    }
}

// Usage
const listener = new DSEventListener(sessionToken, 'callScene');
listener.start();
```

### Python

```python
import requests
import time

class DSEventListener:
    def __init__(self, token, event_name, dss_host="192.168.1.10"):
        self.token = token
        self.event_name = event_name
        self.dss_host = dss_host
        self.subscription_id = 42
        
    def start(self):
        # Subscribe
        self._request(
            "/event/subscribe",
            {"subscriptionID": self.subscription_id, "name": self.event_name}
        )
        
        # Start long-polling loop
        while True:
            try:
                self.long_poll()
            except Exception as e:
                print(f"Error: {e}")
                time.sleep(5)
    
    def long_poll(self):
        result = self._request(
            "/event/get",
            {"subscriptionID": self.subscription_id, "timeout": 60000}
        )
        
        if "events" in result:
            for event in result["events"]:
                self.handle_event(event)
    
    def handle_event(self, event):
        print(f"Event: {event['name']}")
        
        if event["name"] == "callScene":
            scene_id = event["properties"]["sceneID"]
            zone_id = event["source"]["zoneID"]
            print(f"Scene {scene_id} called in zone {zone_id}")
    
    def _request(self, path, params):
        params["token"] = self.token
        resp = requests.get(
            f"https://{self.dss_host}:8080/json{path}",
            params=params,
            verify=False
        )
        return resp.json()["result"]

# Usage
listener = DSEventListener(session_token, "callScene")
listener.start()
```

## Best Practices

1. **Unique Subscription IDs**: Use unique IDs for each subscription to avoid conflicts
2. **Immediate Re-polling**: Don't delay between long-poll requests
3. **Error Handling**: Implement retry logic with backoff for network errors
4. **Timeout Selection**: 60000ms (60s) is recommended for balance between responsiveness and connection overhead
5. **Event Processing**: Handle events quickly to avoid blocking the polling loop
6. **Multiple Events**: Subscribe to multiple event types using different subscription IDs
7. **Reconnection**: Re-subscribe after connection loss or session expiration

## Troubleshooting

**Events Not Received:**
- Verify subscription was successful (`ok: true`)
- Check that events are actually occurring in the system
- Ensure long-poll loop is running continuously
- Verify timeout is reasonable (not too short)

**Duplicate Events:**
- May occur if multiple subscriptions exist for the same event
- Use unique subscription IDs
- Clean up old subscriptions if reconnecting

**Connection Timeouts:**
- Normal behavior if no events occur within timeout period
- Continue polling immediately after timeout

## Notes

- Event subscriptions are tied to the session token
- If session expires, subscriptions are lost
- Some events may be filtered based on user permissions
- Event availability depends on system configuration
- Long-polling is more efficient than frequent status polling
- Consider websocket alternatives if available in newer API versions

## Migration Notes (v1.x to v2.x)

In the transition from API v1.x to v2.x:
- **Scene events**: No longer propagated in the same way
- **Button events**: Removed or significantly changed
- **Alternative**: Monitor output value changes to infer scene activations
- **Workaround**: Set outputs to specific values (e.g., 99% vs 100%) to distinguish scenes
