# Implementation Guide

## Overview

This guide provides practical, step-by-step instructions for implementing a vDC host. It includes code examples, best practices, and common patterns.

## Prerequisites

Before starting implementation:

1. **Read the documentation**:
   - [Getting Started](./01-getting-started.md)
   - [Core Concepts](./02-core-concepts.md)
   - [Protocol Basics](./03-protocol-basics.md)
   - [Property System](./04-property-system.md) - **Essential!**

2. **Choose your technology stack**:
   - Programming language with protobuf support
   - TCP socket library
   - Async I/O framework (recommended)
   - mDNS/Avahi library

3. **Understand your target devices**:
   - Device API/protocol
   - Available features
   - State model
   - Event mechanisms

## Architecture Design

### Component Structure

```
┌─────────────────────────────────────────────┐
│           vDC Host Application              │
├─────────────────────────────────────────────┤
│  ┌────────────────────────────────────┐    │
│  │    Protocol Layer                   │    │
│  │  - TCP Server                       │    │
│  │  - Protobuf Encoding/Decoding      │    │
│  │  - Message Routing                  │    │
│  └────────────────────────────────────┘    │
│  ┌────────────────────────────────────┐    │
│  │    Session Management               │    │
│  │  - Connection Handling              │    │
│  │  - Keep-Alive (Ping/Pong)          │    │
│  │  - State Machine                    │    │
│  └────────────────────────────────────┘    │
│  ┌────────────────────────────────────┐    │
│  │    vDC Manager                      │    │
│  │  - vDC Registry                     │    │
│  │  - Device Announcement              │    │
│  │  - Property Tree Manager            │    │
│  └────────────────────────────────────┘    │
│  ┌────────────────────────────────────┐    │
│  │    Device Adapters                  │    │
│  │  - External API Client              │    │
│  │  - State Synchronization            │    │
│  │  - Event Handling                   │    │
│  └────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
         │                      ▲
         │ Commands             │ Events/State
         ▼                      │
┌─────────────────────────────────────────────┐
│       External Devices/Systems              │
│  (Hue, Sonos, MQTT, HTTP, etc.)            │
└─────────────────────────────────────────────┘
```

### Class Structure Example (Python)

```python
class VdcHost:
    """Main vDC host application"""
    def __init__(self):
        self.dsuid = generate_host_dsuid()
        self.vdcs = {}
        self.server = None
        self.sessions = []
    
    def start(self):
        """Start the vDC host"""
        self.server = TcpServer(port=8440)
        self.server.on_connection = self.handle_new_connection
        self.announce_service()  # mDNS
        
    def handle_new_connection(self, socket):
        """Handle new vdSM connection"""
        session = VdcSession(socket, self)
        self.sessions.append(session)

class VdcSession:
    """Manages a single vdSM connection"""
    def __init__(self, socket, host):
        self.socket = socket
        self.host = host
        self.state = "CONNECTED"
        self.message_id_counter = 1
        
    def handle_message(self, message):
        """Route incoming messages"""
        if message.type == VDSM_REQUEST_HELLO:
            self.handle_hello(message)
        elif message.type == VDSM_REQUEST_GET_PROPERTY:
            self.handle_get_property(message)
        # ... etc

class Vdc:
    """Represents a virtual device connector"""
    def __init__(self, dsuid, name):
        self.dsuid = dsuid
        self.name = name
        self.devices = {}
        self.properties = self.build_properties()
    
    def build_properties(self):
        """Build vDC property tree"""
        return {
            "name": self.name,
            "dSUID": self.dsuid,
            "modelName": "MyVdc",
            # ... more properties
        }

class VirtualDevice:
    """Represents a virtual dS device"""
    def __init__(self, dsuid, vdc):
        self.dsuid = dsuid
        self.vdc = vdc
        self.properties = self.build_properties()
    
    def build_properties(self):
        """Build device property tree"""
        return {
            "name": "Device Name",
            "dSUID": self.dsuid,
            # ... more properties
        }
    
    def call_scene(self, scene_number):
        """Handle scene call"""
        pass
    
    def set_channel_value(self, channel_id, value):
        """Set output channel value"""
        pass
```

## Step-by-Step Implementation

### Step 1: Set Up Basic TCP Server

```python
import socket
import threading
import struct

class TcpServer:
    def __init__(self, port=8440):
        self.port = port
        self.socket = None
        self.running = False
        
    def start(self):
        """Start listening for connections"""
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.socket.bind(('0.0.0.0', self.port))
        self.socket.listen(5)
        self.running = True
        
        print(f"Listening on port {self.port}")
        
        while self.running:
            client_socket, address = self.socket.accept()
            print(f"Connection from {address}")
            
            # Handle in separate thread
            thread = threading.Thread(
                target=self.handle_client,
                args=(client_socket,)
            )
            thread.start()
    
    def handle_client(self, socket):
        """Handle client connection"""
        try:
            while True:
                message = self.receive_message(socket)
                if not message:
                    break
                    
                # Process message
                response = self.process_message(message)
                if response:
                    self.send_message(socket, response)
        finally:
            socket.close()
    
    def send_message(self, sock, message):
        """Send protobuf message with length prefix"""
        data = message.SerializeToString()
        length = len(data)
        sock.sendall(struct.pack('<I', length))
        sock.sendall(data)
    
    def receive_message(self, sock):
        """Receive protobuf message with length prefix"""
        # Read 4-byte length prefix
        length_data = sock.recv(4)
        if not length_data or len(length_data) < 4:
            return None
        
        length = struct.unpack('<I', length_data)[0]
        
        # Read message data
        data = b''
        while len(data) < length:
            chunk = sock.recv(min(4096, length - len(data)))
            if not chunk:
                return None
            data += chunk
        
        # Parse protobuf message
        from vdc_pb2 import Message  # Generated from .proto
        message = Message()
        message.ParseFromString(data)
        return message
```

### Step 2: Implement Protocol Buffer Support

1. **Generate Python code from .proto**:
```bash
protoc --python_out=. genericVDC.proto
```

This creates `genericVDC_pb2.py` with all message classes.

2. **Use in your code**:
```python
from genericVDC_pb2 import *

# Create a message
hello_response = Message()
hello_response.type = VDC_RESPONSE_HELLO
hello_response.message_id = request_message_id
hello_response.vdc_response_hello.dSUID = my_dsuid

# Serialize
data = hello_response.SerializeToString()

# Parse
received_message = Message()
received_message.ParseFromString(data)
```

### Step 3: Implement Hello Handshake

```python
def handle_hello(self, message):
    """Handle vdsm_RequestHello"""
    request = message.vdsm_request_hello
    
    print(f"Received Hello from vdSM: {request.dSUID}")
    print(f"API version: {request.api_version}")
    
    # Validate API version
    if request.api_version not in [2, 3]:
        # Send error response
        error_response = Message()
        error_response.type = GENERIC_RESPONSE
        error_response.message_id = message.message_id
        error_response.generic_response.code = ERR_INCOMPATIBLE_API
        error_response.generic_response.description = "API version not supported"
        return error_response
    
    # Send hello response
    response = Message()
    response.type = VDC_RESPONSE_HELLO
    response.message_id = message.message_id
    response.vdc_response_hello.dSUID = self.host.dsuid
    
    self.state = "HELLO_COMPLETE"
    
    # After hello, announce vDCs
    self.announce_vdcs()
    
    return response
```

### Step 4: Implement vDC and Device Announcement

```python
def announce_vdcs(self):
    """Announce all vDCs"""
    for vdc in self.host.vdcs.values():
        message = Message()
        message.type = VDC_SEND_ANNOUNCE_VDC
        message.message_id = 0  # No response expected
        message.vdc_send_announce_vdc.dSUID = vdc.dsuid
        
        self.send_message(message)
        
        # Announce devices for this vDC
        self.announce_devices(vdc)

def announce_devices(self, vdc):
    """Announce all devices for a vDC"""
    for device in vdc.devices.values():
        message = Message()
        message.type = VDC_SEND_ANNOUNCE_DEVICE
        message.message_id = 0
        message.vdc_send_announce_device.dSUID = device.dsuid
        message.vdc_send_announce_device.vdc_dSUID = vdc.dsuid
        
        self.send_message(message)
```

### Step 5: Implement Property System

```python
class PropertyTreeManager:
    """Manages property tree access"""
    
    def get_property(self, dsuid, query):
        """Handle GetProperty request"""
        # Find entity (vDC or device)
        entity = self.find_entity(dsuid)
        if not entity:
            return self.error_response(ERR_NOT_FOUND, f"Entity {dsuid} not found")
        
        # Process query
        result = []
        for query_elem in query:
            prop = self.resolve_query(entity.properties, query_elem)
            result.append(prop)
        
        return result
    
    def resolve_query(self, properties, query_elem):
        """Resolve a single query element"""
        name = query_elem.name
        
        if name not in properties:
            # Property doesn't exist
            return None
        
        value = properties[name]
        result = PropertyElement()
        result.name = name
        
        # If query has no sub-elements, return all
        if not query_elem.elements:
            self.set_property_value(result, value)
        else:
            # Process nested query
            for sub_query in query_elem.elements:
                sub_result = self.resolve_query(value, sub_query)
                if sub_result:
                    result.elements.append(sub_result)
        
        return result
    
    def set_property_value(self, elem, value):
        """Set PropertyElement value based on Python type"""
        if isinstance(value, bool):
            elem.value.v_bool = value
        elif isinstance(value, str):
            elem.value.v_string = value
        elif isinstance(value, int):
            elem.value.v_uint64 = value
        elif isinstance(value, float):
            elem.value.v_double = value
        elif isinstance(value, dict):
            # Container - add all children
            for key, val in value.items():
                child = elem.elements.add()
                child.name = key
                self.set_property_value(child, val)
        elif isinstance(value, list):
            # Array - add indexed children
            for idx, val in enumerate(value):
                child = elem.elements.add()
                child.name = str(idx)
                self.set_property_value(child, val)
```

### Step 6: Implement Scene Operations

```python
def handle_call_scene(self, message):
    """Handle scene call notification"""
    notification = message.vdsm_send_call_scene
    
    scene_number = notification.scene
    force = notification.force
    
    for dsuid in notification.dSUID:
        device = self.find_device(dsuid)
        if device:
            device.call_scene(scene_number, force)
    
    # No response for notifications

class VirtualDevice:
    def call_scene(self, scene_number, force=False):
        """Apply scene to device"""
        # Get scene configuration
        scene_config = self.get_scene_config(scene_number)
        
        if not scene_config:
            print(f"Scene {scene_number} not configured")
            return
        
        # Apply channel values from scene
        for channel_id, value in scene_config.items():
            self.set_channel_value(channel_id, value, apply_now=True)
```

### Step 7: Implement Keep-Alive

```python
def handle_ping(self, message):
    """Handle ping request"""
    request = message.vdsm_send_ping
    
    # Send pong response
    response = Message()
    response.type = VDC_SEND_PONG
    response.message_id = 0  # Ping/pong don't use message_id correlation
    response.vdc_send_pong.dSUID = self.host.dsuid
    
    return response

# In session management:
def check_connection_health(self):
    """Monitor connection health"""
    # If no message received for > 60 seconds, consider connection dead
    if time.time() - self.last_message_time > 60:
        self.close_connection()
```

### Step 8: Implement mDNS Service Advertisement

Using python-zeroconf:

```python
from zeroconf import ServiceInfo, Zeroconf
import socket

class ServiceAdvertiser:
    def __init__(self, port=8440):
        self.port = port
        self.zeroconf = Zeroconf()
        
    def advertise(self):
        """Advertise vDC host service via mDNS"""
        hostname = socket.gethostname()
        
        info = ServiceInfo(
            "_ds-vdc._tcp.local.",
            f"MyVdcHost._ds-vdc._tcp.local.",
            addresses=[socket.inet_aton(self.get_ip())],
            port=self.port,
            properties={
                'vdchost': 'my-vdc-host',
                'model': 'MyVdcHost/1.0'
            }
        )
        
        self.zeroconf.register_service(info)
        print(f"Service advertised on port {self.port}")
    
    def get_ip(self):
        """Get local IP address"""
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        try:
            s.connect(('8.8.8.8', 80))
            ip = s.getsockname()[0]
        finally:
            s.close()
        return ip
```

## Complete Example: Simple Light vDC

```python
#!/usr/bin/env python3
"""
Simple Light vDC Example
Demonstrates a minimal vDC host with mock light devices
"""

import uuid
from genericVDC_pb2 import *

class SimpleLightVdc:
    """Simple vDC for mock light devices"""
    
    def __init__(self):
        self.dsuid = self.generate_dsuid("light-vdc")
        self.devices = {}
        
        # Create some mock devices
        self.add_device("living-room", "Living Room Light")
        self.add_device("bedroom", "Bedroom Light")
    
    def generate_dsuid(self, identifier):
        """Generate deterministic dSUID"""
        namespace = uuid.UUID('12345678-1234-5678-1234-567812345678')
        device_uuid = uuid.uuid5(namespace, identifier)
        dsuid = device_uuid.bytes + b'\x00'
        return dsuid.hex().upper()
    
    def add_device(self, id, name):
        """Add a light device"""
        device = SimpleLightDevice(
            dsuid=self.generate_dsuid(f"device-{id}"),
            name=name,
            vdc=self
        )
        self.devices[device.dsuid] = device
    
    def get_properties(self):
        """Get vDC properties"""
        return {
            "name": "Simple Light vDC",
            "dSUID": self.dsuid,
            "modelName": "SimpleLightVdc",
            "modelVersion": "1.0",
            "vendorName": "MyCompany",
            "capabilities": {
                "metering": False,
                "identification": False
            }
        }

class SimpleLightDevice:
    """Simple light device"""
    
    def __init__(self, dsuid, name, vdc):
        self.dsuid = dsuid
        self.name = name
        self.vdc = vdc
        self.brightness = 0.0  # 0-100
        self.scenes = self.default_scenes()
    
    def default_scenes(self):
        """Default scene configurations"""
        return {
            0: {"brightness": 0.0},      # Off
            5: {"brightness": 100.0},    # Scene 1: Full brightness
            17: {"brightness": 50.0},    # Scene 2: Half brightness
            18: {"brightness": 25.0},    # Scene 3: Low brightness
        }
    
    def get_properties(self):
        """Get device properties"""
        return {
            "name": self.name,
            "dSUID": self.dsuid,
            "model": "Virtual Light",
            "modelVersion": "1.0",
            "primaryGroup": 1,  # Yellow (light)
            "output": {
                "channels": [
                    {
                        "channelType": 1,  # Brightness
                        "channelId": "brightness",
                        "name": "Brightness",
                        "min": 0.0,
                        "max": 100.0,
                        "resolution": 0.1
                    }
                ]
            }
        }
    
    def call_scene(self, scene_number):
        """Apply scene"""
        if scene_number in self.scenes:
            config = self.scenes[scene_number]
            self.brightness = config["brightness"]
            print(f"{self.name}: Scene {scene_number} -> brightness={self.brightness}%")
        else:
            print(f"{self.name}: Unknown scene {scene_number}")
    
    def set_channel_value(self, channel_id, value):
        """Set channel value"""
        if channel_id == "brightness":
            self.brightness = max(0.0, min(100.0, value))
            print(f"{self.name}: Brightness set to {self.brightness}%")
    
    def dim_channel(self, channel_id, direction):
        """Dim channel"""
        if channel_id == "brightness":
            step = 10.0 if direction == "up" else -10.0
            self.brightness = max(0.0, min(100.0, self.brightness + step))
            print(f"{self.name}: Dimmed to {self.brightness}%")

# Usage:
# vdc = SimpleLightVdc()
# host = VdcHost()
# host.add_vdc(vdc)
# host.start()
```

## Best Practices

### 1. Error Handling

Always handle errors gracefully:

```python
try:
    result = external_api.set_brightness(value)
except ConnectionError:
    return GenericResponse(
        code=ERR_SERVICE_NOT_AVAILABLE,
        description="Device offline"
    )
except ValueError:
    return GenericResponse(
        code=ERR_INVALID_VALUE_TYPE,
        description="Invalid brightness value"
    )
```

### 2. Logging

Implement comprehensive logging:

```python
import logging

logger = logging.getLogger(__name__)

def handle_message(self, message):
    logger.debug(f"Received: {message.type}")
    # Process message
    logger.debug(f"Sent response: {response.type}")
```

### 3. State Persistence

Persist important state:

```python
import json

class StateManager:
    def save_scenes(self, device):
        """Save device scenes"""
        data = {
            'dsuid': device.dsuid,
            'scenes': device.scenes
        }
        with open(f'scenes_{device.dsuid}.json', 'w') as f:
            json.dump(data, f)
    
    def load_scenes(self, device):
        """Load device scenes"""
        try:
            with open(f'scenes_{device.dsuid}.json', 'r') as f:
                data = json.load(f)
                return data['scenes']
        except FileNotFoundError:
            return device.default_scenes()
```

### 4. Async I/O

Use async for better performance:

```python
import asyncio

class AsyncVdcHost:
    async def handle_client(self, reader, writer):
        """Handle client with asyncio"""
        while True:
            # Read message
            message = await self.receive_message(reader)
            if not message:
                break
            
            # Process
            response = await self.process_message(message)
            
            # Send response
            if response:
                await self.send_message(writer, response)
```

## Testing

### Unit Tests

```python
import unittest

class TestPropertyTree(unittest.TestCase):
    def test_get_simple_property(self):
        tree = {"name": "Test Device"}
        manager = PropertyTreeManager()
        
        query = PropertyElement()
        query.name = "name"
        
        result = manager.resolve_query(tree, query)
        self.assertEqual(result.value.v_string, "Test Device")
    
    def test_get_nested_property(self):
        tree = {
            "device": {
                "model": "Test Model"
            }
        }
        # ... test nested access
```

### Integration Tests

Test with mock vdSM:

```python
class MockVdsm:
    """Mock vdSM for testing"""
    
    def connect(self, host, port):
        self.socket = socket.connect((host, port))
    
    def send_hello(self):
        message = Message()
        message.type = VDSM_REQUEST_HELLO
        message.vdsm_request_hello.dSUID = "MOCK_VDSM"
        message.vdsm_request_hello.api_version = 3
        self.send(message)
        
        response = self.receive()
        assert response.type == VDC_RESPONSE_HELLO
```

## Deployment

### Running as Service

SystemD service file:

```ini
[Unit]
Description=My vDC Host
After=network.target

[Service]
Type=simple
User=vdc
WorkingDirectory=/opt/my-vdc-host
ExecStart=/usr/bin/python3 /opt/my-vdc-host/main.py
Restart=always

[Install]
WantedBy=multi-user.target
```

### Docker Deployment

```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8440
CMD ["python", "main.py"]
```

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| vdSM doesn't discover host | Check mDNS service, firewall |
| Connection drops | Implement ping/pong properly |
| Properties not found | Verify property tree structure |
| Scenes don't work | Check scene number mapping |

## Next Steps

- Study the [original documentation](../originalDocFiles/) for detailed specifications
- Review [Property System](./04-property-system.md) again - it's critical!
- Implement incrementally - start small
- Test thoroughly with real dSS if possible

---

[Back to Documentation Index](./README.md)
