# Quick Start Guide

## Introduction

This guide will help you get started with the digitalSTROM vDC-API quickly. Whether you're building a new vDC host or integrating with an existing one, this guide provides the essential steps and code examples.

> **Note:** Code examples in this guide are simplified for clarity and educational purposes. In production code, follow best practices such as placing all imports at the top of files, adding comprehensive error handling, and implementing proper logging.

## Prerequisites

Before you begin:

1. **Understanding of digitalSTROM:** Familiarity with the digitalSTROM ecosystem
2. **Protocol Buffers:** Basic knowledge of Google Protocol Buffers
3. **Network Programming:** Experience with TCP/IP socket programming
4. **Development Environment:** C, C++, Python, or other language with protobuf support

## Choose Your Approach

### Option 1: Use Existing Frameworks (Recommended)

**libdsvdc (C library)**
- Best for: Device vDCs (Types 1 & 2)
- Language: C
- License: GPL v3 / Commercial

**vdcd (C++ framework)**
- Best for: Gateway vDCs (Type 3)
- Language: C++
- License: GPL v3 / Commercial

### Option 2: Implement from Scratch

Follow this guide to implement the vDC-API directly using Protocol Buffers and TCP sockets.

## Implementation Steps

### Step 1: Set Up Protocol Buffers

**1.1 Copy the Protocol Buffer Schema**

```bash
# Copy genericVDC.proto to your project
cp originalDocFiles/genericVDC.proto src/
```

**1.2 Generate Code for Your Language**

```bash
# For Python
protoc --python_out=. genericVDC.proto

# For C++
protoc --cpp_out=. genericVDC.proto

# For Java
protoc --java_out=. genericVDC.proto
```

### Step 2: Implement Service Discovery (Avahi)

**2.1 Create Avahi Service File**

Create `/etc/avahi/services/ds-vdc.service`:

```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">digitalSTROM vDC host on %h</name>
  <service protocol="ipv4">
    <type>_ds-vdc._tcp</type>
    <port>8444</port>
  </service>
</service-group>
```

**2.2 Install and Enable Avahi**

```bash
# Ubuntu/Debian
sudo apt-get install avahi-daemon
sudo systemctl enable avahi-daemon
sudo systemctl start avahi-daemon
```

### Step 3: Implement TCP Server

**3.1 Create TCP Server (Python Example)**

```python
import socket
import struct
from genericVDC_pb2 import Message

class VdcServer:
    def __init__(self, port=8444):
        self.port = port
        self.sock = None
        self.dsuid = "199500440F80000000000F8001EXAMPLE"
        self.current_session = None
        self.devices = {}
        
    def start(self):
        """Start the vDC server"""
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.sock.bind(('', self.port))
        self.sock.listen(1)
        print(f"vDC server listening on port {self.port}")
        
        while True:
            conn, addr = self.sock.accept()
            print(f"Connection from {addr}")
            self.handle_client(conn)
    
    def handle_client(self, conn):
        """Handle client connection"""
        try:
            while True:
                msg = self.receive_message(conn)
                if msg is None:
                    break
                response = self.process_message(msg)
                if response:
                    self.send_message(conn, response)
        except Exception as e:
            print(f"Error: {e}")
        finally:
            conn.close()
    
    def receive_message(self, conn):
        """Receive a Protocol Buffer message"""
        # Read 2-byte header
        header = conn.recv(2)
        if len(header) < 2:
            return None
        
        # Parse length (big-endian)
        length = struct.unpack('!H', header)[0]
        
        # Read payload
        payload = b''
        while len(payload) < length:
            chunk = conn.recv(length - len(payload))
            if not chunk:
                return None
            payload += chunk
        
        # Deserialize
        message = Message()
        message.ParseFromString(payload)
        return message
    
    def send_message(self, conn, message):
        """Send a Protocol Buffer message"""
        # Serialize
        payload = message.SerializeToString()
        
        # Check size
        if len(payload) > 16384:
            raise ValueError("Message too large")
        
        # Create header (big-endian)
        header = struct.pack('!H', len(payload))
        
        # Send
        conn.sendall(header + payload)
    
    def process_message(self, msg):
        """Process received message and return response"""
        if msg.type == Message.VDSM_REQUEST_HELLO:
            return self.handle_hello(msg)
        elif msg.type == Message.VDSM_REQUEST_GET_PROPERTY:
            return self.handle_get_property(msg)
        elif msg.type == Message.VDSM_REQUEST_SET_PROPERTY:
            return self.handle_set_property(msg)
        # Add more handlers...
        return None
    
    def error_response(self, message_id, code, description):
        """Create an error response"""
        response = Message()
        response.type = Message.GENERIC_RESPONSE
        response.message_id = message_id
        response.generic_response.code = code
        response.generic_response.description = description
        return response
    
    def success_response(self, message_id):
        """Create a success response"""
        response = Message()
        response.type = Message.GENERIC_RESPONSE
        response.message_id = message_id
        response.generic_response.code = Message.ERR_OK
        return response
    
    def get_device(self, dsuid):
        """Get device by dSUID"""
        return self.devices.get(dsuid)
    
    def handle_hello(self, msg):
        """Handle hello request"""
        hello_req = msg.vdsm_request_hello
        
        # Check API version
        if hello_req.api_version != 2:
            # Return error
            response = Message()
            response.type = Message.GENERIC_RESPONSE
            response.message_id = msg.message_id
            response.generic_response.code = Message.ERR_INCOMPATIBLE_API
            response.generic_response.description = "API version not supported"
            return response
        
        # Return success
        response = Message()
        response.type = Message.VDC_RESPONSE_HELLO
        response.message_id = msg.message_id
        response.vdc_response_hello.dSUID = self.dsuid
        return response

if __name__ == '__main__':
    server = VdcServer()
    server.start()
```

### Step 4: Implement Session Management

**4.1 Handle Hello Exchange**

```python
def handle_hello(self, msg):
    """Handle hello request and start session"""
    hello_req = msg.vdsm_request_hello
    
    # Check if already connected
    if self.current_session:
        return self.error_response(
            msg.message_id,
            Message.ERR_SERVICE_NOT_AVAILABLE,
            "Already connected to another vdSM"
        )
    
    # Check API version
    if hello_req.api_version not in [2, 3]:
        return self.error_response(
            msg.message_id,
            Message.ERR_INCOMPATIBLE_API,
            f"API version {hello_req.api_version} not supported"
        )
    
    # Create session
    self.current_session = {
        'vdsm_dsuid': hello_req.dSUID,
        'api_version': hello_req.api_version
    }
    
    # Return hello response
    response = Message()
    response.type = Message.VDC_RESPONSE_HELLO
    response.message_id = msg.message_id
    response.vdc_response_hello.dSUID = self.dsuid
    return response
```

**4.2 Implement Bye Handler**

```python
def handle_bye(self, msg):
    """Handle session termination"""
    self.current_session = None
    
    response = Message()
    response.type = Message.GENERIC_RESPONSE
    response.message_id = msg.message_id
    response.generic_response.code = Message.ERR_OK
    return response
```

### Step 5: Announce vDC and Devices

**5.1 Announce vDC**

```python
def announce_vdc(self, conn, vdc_dsuid):
    """Announce a logical vDC"""
    msg = Message()
    msg.type = Message.VDC_SEND_ANNOUNCE_VDC
    msg.vdc_send_announce_vdc.dSUID = vdc_dsuid
    
    self.send_message(conn, msg)
    
    # Wait for response
    response = self.receive_message(conn)
    if response.generic_response.code == Message.ERR_OK:
        print(f"vDC {vdc_dsuid} announced successfully")
    else:
        print(f"Failed to announce vDC: {response.generic_response.description}")
```

**5.2 Announce Device**

```python
def announce_device(self, conn, device_dsuid, vdc_dsuid):
    """Announce a device"""
    msg = Message()
    msg.type = Message.VDC_SEND_ANNOUNCE_DEVICE
    msg.vdc_send_announce_device.dSUID = device_dsuid
    msg.vdc_send_announce_device.vdc_dSUID = vdc_dsuid
    
    self.send_message(conn, msg)
    
    # Wait for response
    response = self.receive_message(conn)
    if response.generic_response.code == Message.ERR_OK:
        print(f"Device {device_dsuid} announced successfully")
    else:
        print(f"Failed to announce device: {response.generic_response.description}")
```

### Step 6: Implement Property Access

**6.1 Handle GetProperty**

```python
def handle_get_property(self, msg):
    """Handle property read request"""
    req = msg.vdsm_request_get_property
    device_dsuid = req.dSUID
    
    # Get device
    device = self.get_device(device_dsuid)
    if not device:
        return self.error_response(
            msg.message_id,
            Message.ERR_NOT_FOUND,
            f"Device {device_dsuid} not found"
        )
    
    # Process query
    properties = self.process_property_query(device, req.query)
    
    # Return response
    response = Message()
    response.type = Message.VDC_RESPONSE_GET_PROPERTY
    response.message_id = msg.message_id
    response.vdc_response_get_property.properties.extend(properties)
    return response

def process_property_query(self, device, query):
    """Process property query and return matching properties"""
    result = []
    for query_elem in query:
        if hasattr(device, 'get_property'):
            prop_value = device.get_property(query_elem.name)
            if prop_value:
                from genericVDC_pb2 import PropertyElement
                prop = PropertyElement()
                prop.name = query_elem.name
                prop.value.CopyFrom(prop_value)
                result.append(prop)
    return result
```

**6.2 Handle SetProperty**

```python
def handle_set_property(self, msg):
    """Handle property write request"""
    req = msg.vdsm_request_set_property
    device_dsuid = req.dSUID
    
    # Get device
    device = self.get_device(device_dsuid)
    if not device:
        return self.error_response(
            msg.message_id,
            Message.ERR_NOT_FOUND,
            f"Device {device_dsuid} not found"
        )
    
    # Apply properties
    try:
        for prop in req.properties:
            device.set_property(prop.name, prop.value)
    except Exception as e:
        return self.error_response(
            msg.message_id,
            Message.ERR_INVALID_VALUE_TYPE,
            str(e)
        )
    
    # Return success
    return self.success_response(msg.message_id)
```

### Step 7: Implement Actions and Notifications

**7.1 Handle Call Scene**

```python
def handle_call_scene(self, msg):
    """Handle scene call notification"""
    notif = msg.vdsm_send_call_scene
    
    for device_dsuid in notif.dSUID:
        device = self.get_device(device_dsuid)
        if device:
            device.call_scene(notif.scene, notif.force)
    
    # Notifications don't expect responses
    return None
```

**7.2 Handle Dim Channel**

```python
def handle_dim_channel(self, msg):
    """Handle channel dimming"""
    notif = msg.vdsm_send_dim_channel
    
    for device_dsuid in notif.dSUID:
        device = self.get_device(device_dsuid)
        if device:
            if notif.mode == 1:
                device.start_dimming_up(notif.channel)
            elif notif.mode == -1:
                device.start_dimming_down(notif.channel)
            else:
                device.stop_dimming(notif.channel)
    
    return None
```

### Step 8: Implement Push Notifications

**8.1 Send Property Change**

```python
def push_property_change(self, conn, device_dsuid, property_name, value):
    """Push property change to vdSM"""
    msg = Message()
    msg.type = Message.VDC_SEND_PUSH_NOTIFICATION
    msg.vdc_send_push_notification.dSUID = device_dsuid
    
    # Create property element
    prop = msg.vdc_send_push_notification.changedproperties.add()
    prop.name = property_name
    prop.value.CopyFrom(value)
    
    self.send_message(conn, msg)
```

**8.2 Push Button Event**

```python
def push_button_event(self, conn, device_dsuid, button_index, button_state):
    """Push button state change"""
    from genericVDC_pb2 import PropertyValue
    
    value = PropertyValue()
    value.v_uint64 = button_state
    
    self.push_property_change(
        conn,
        device_dsuid,
        f"buttonInputStates.{button_index}.value",
        value
    )
```

## Complete Example: Simple Light Device

```python
class SimpleLightDevice:
    def __init__(self, dsuid, name):
        self.dsuid = dsuid
        self.name = name
        self.brightness = 0
        self.is_on = False
    
    def call_scene(self, scene_number, force=False):
        """Handle scene call"""
        # Scene 5 = preset 1, Scene 14 = off
        if scene_number == 5:
            self.set_brightness(100)
        elif scene_number == 14:
            self.set_brightness(0)
    
    def set_brightness(self, value):
        """Set light brightness"""
        self.brightness = max(0, min(100, value))
        self.is_on = self.brightness > 0
        print(f"Light {self.name}: brightness={self.brightness}%, on={self.is_on}")
    
    def start_dimming_up(self, channel=0):
        """Start dimming up"""
        print(f"Light {self.name}: starting dim up")
        # Implement dimming logic
    
    def start_dimming_down(self, channel=0):
        """Start dimming down"""
        print(f"Light {self.name}: starting dim down")
        # Implement dimming logic
    
    def stop_dimming(self, channel=0):
        """Stop dimming"""
        print(f"Light {self.name}: stopping dim")
    
    def get_property(self, name):
        """Get property value"""
        if name == "name":
            return PropertyValue(v_string=self.name)
        elif name == "outputStates.0.value":
            return PropertyValue(v_double=self.brightness)
        # Add more properties...
    
    def set_property(self, name, value):
        """Set property value"""
        if name == "name":
            self.name = value.v_string
        elif name == "outputStates.0.value":
            self.set_brightness(value.v_double)
```

## Testing Your Implementation

### 1. Test Service Discovery

```bash
# Check if service is announced
avahi-browse -a

# Should show: _ds-vdc._tcp
```

### 2. Test TCP Connection

```bash
# Connect with telnet
telnet localhost 8444
```

### 3. Test with Python Client

```python
# Simple client test
import socket

sock = socket.socket()
sock.connect(('localhost', 8444))

# Send hello (you'll need to construct proper protobuf message)
# ...

sock.close()
```

## Next Steps

1. **Read Core Documentation:**
   - [API Basics](api-basics.md)
   - [Session Management](session-management.md)
   - [Device Operations](device-operations.md)

2. **Implement Properties:**
   - [Properties](properties.md)
   - Define your device's property schema

3. **Add Device Actions:**
   - [Actions and Notifications](actions-notifications.md)
   - Implement scene support, dimming, etc.

4. **Error Handling:**
   - [Error Handling](error-handling.md)
   - Implement robust error recovery

5. **Production Readiness:**
   - Add logging
   - Implement reconnection logic
   - Add persistent storage for configuration
   - Test with real digitalSTROM system

## Common Issues and Solutions

### Issue: Avahi service not visible
**Solution:** Check avahi-daemon is running, verify service file syntax

### Issue: Connection refused
**Solution:** Check port is not in use, verify firewall settings

### Issue: ERR_INCOMPATIBLE_API
**Solution:** Check API version in hello request matches server support

### Issue: Messages not parsed
**Solution:** Verify Protocol Buffer schema matches, check message framing

## Resources

- **Original Documentation:** `originalDocFiles/` directory
- **Protocol Schema:** `originalDocFiles/genericVDC.proto`
- **Community Resources:** digitalSTROM developer website
- **Sample Code:** libdsvdc and vdcd repositories

## Support

For additional help:
1. Review the complete documentation in the `docs/` directory
2. Check the original PDF documentation
3. Consult digitalSTROM developer resources
4. Join digitalSTROM developer community
