# Getting Started with vDC API

## Introduction

The digitalSTROM Virtual Device Connector (vDC) API enables integration of third-party devices into the digitalSTROM ecosystem. This guide will help you understand the basics and get started with implementing a vDC host.

## What You'll Learn

- Core concepts and terminology
- Architecture overview
- How devices communicate with the digitalSTROM Server
- First steps in implementation

## Core Concepts

### The digitalSTROM System

digitalSTROM is a home automation system that uses building wiring to communicate with devices. The system consists of:

- **dSS (digitalSTROM Server)**: Central controller managing the entire system
- **dSM (digitalSTROM Meter)**: Hardware modules connected to devices
- **dSD (digitalSTROM Devices)**: Individual devices in the system

### Virtual Device Integration

The vDC API allows external devices to integrate without requiring physical digitalSTROM hardware:

```
┌─────────────────────────────────────────────────────────────┐
│                digitalSTROM Server (dSS)                     │
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │   vdSM   │  │   vdSM   │  │   vdSM   │  │   dSM    │    │
│  │ Virtual  │  │ Virtual  │  │ Virtual  │  │ Physical │    │
│  └─────┬────┘  └─────┬────┘  └─────┬────┘  └─────┬────┘    │
└────────┼─────────────┼─────────────┼─────────────┼──────────┘
         │             │             │             │
         │             │             │             │
    ┌────▼────┐   ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
    │vDC Host │   │vDC Host │   │vDC Host │   │Physical │
    │         │   │         │   │         │   │Devices  │
    │┌───────┐│   │┌───────┐│   │┌───────┐│   └─────────┘
    ││ vDC   ││   ││ vDC   ││   ││ vDC   ││
    ││ (Hue) ││   ││(Sonos)││   ││(MQTT) ││
    │└───┬───┘│   │└───┬───┘│   │└───┬───┘│
    │    │    │   │    │    │   │    │    │
    │ ┌──▼──┐ │   │ ┌──▼──┐ │   │ ┌──▼──┐ │
    │ │vdSD │ │   │ │vdSD │ │   │ │vdSD │ │
    │ │vdSD │ │   │ │vdSD │ │   │ │vdSD │ │
    │ │vdSD │ │   │ │vdSD │ │   │ │vdSD │ │
    │ └─────┘ │   │ └─────┘ │   │ └─────┘ │
    └─────────┘   └─────────┘   └─────────┘
       Hue           Sonos          MQTT
      Lights        Speakers       Devices
```

## Key Terminology

| Term | Full Name | Description |
|------|-----------|-------------|
| **vDC** | Virtual Device Connector | A logical entity representing a class/type of external devices. Has its own dSUID. One vDC per device technology (e.g., one for Philips Hue, one for Sonos). |
| **vDC host** | Virtual Device Connector Host | A network device (software) that implements the vDC protocol and hosts one or more vDCs. Provides a TCP server socket for vdSM connections. |
| **vdSM** | Virtual digitalSTROM Meter | A component in the dSS that connects to vDC hosts via TCP. Manages the connection and communication with vDC hosts. |
| **vdSD** | Virtual digitalSTROM Device | A single device in the dS system. Behaves like a physical digitalSTROM device but is implemented by a vDC host. |
| **dSUID** | digitalSTROM Unique Identifier | A unique identifier for each entity (vDC, device) in the system. |
| **Property Tree** | - | Hierarchical structure for accessing device configuration, state, and capabilities. |

## Architecture Overview

### Components and Relationships

1. **vDC Host** (Your Implementation)
   - Runs as a network service
   - Listens on a TCP port (default: 8440)
   - Can host multiple vDCs
   - Manages connection to external devices (Hue, Sonos, etc.)

2. **vdSM** (Part of dSS)
   - Discovers vDC hosts (via Avahi/mDNS)
   - Initiates TCP connection to vDC host
   - Manages communication protocol
   - Routes messages between dSS and vDC host

3. **vDC** (Logical Grouping)
   - Represents a device technology/class
   - Has its own unique dSUID
   - Contains multiple vdSDs (devices)

4. **vdSD** (Individual Device)
   - Represents one external device
   - Has its own unique dSUID
   - Exposed through property tree
   - Responds to actions and notifications

### Communication Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Communication Flow                        │
└─────────────────────────────────────────────────────────────┘

1. Discovery:
   vDC Host → Avahi/mDNS → vdSM discovers host

2. Connection:
   vdSM → TCP Connect → vDC Host (port 8440)

3. Handshake:
   vdSM → HELLO → vDC Host
   vDC Host → HELLO Response → vdSM

4. vDC Announcement:
   vDC Host → ANNOUNCE_VDC → vdSM (for each vDC)

5. Device Announcement:
   vDC Host → ANNOUNCE_DEVICE → vdSM (for each vdSD)

6. Operation:
   vdSM ⟷ GetProperty/SetProperty ⟷ vDC Host
   vdSM ⟷ Notifications (Scenes, etc.) ⟷ vDC Host
   vDC Host → PushProperty → vdSM (state changes)

7. Keep-Alive:
   vdSM ⟷ PING/PONG ⟷ vDC Host
```

## Protocol Basics

### Message Format

The vDC API uses **Protocol Buffers (protobuf)** for message encoding:

- Binary encoding for efficiency
- Strongly typed messages
- Defined in `genericVDC.proto`
- Messages sent over TCP connection

### Key Message Categories

1. **Session Management**
   - `vdsm_RequestHello` / `vdc_ResponseHello`
   - `vdsm_SendBye`

2. **Property Access**
   - `vdsm_RequestGetProperty` / `vdc_ResponseGetProperty`
   - `vdsm_RequestSetProperty`
   - `vdc_SendPushNotification` (property changes)

3. **Device Lifecycle**
   - `vdc_SendAnnounceVdc`
   - `vdc_SendAnnounceDevice`
   - `vdc_SendVanish`
   - `vdc_SendIdentify`

4. **Actions/Notifications**
   - `vdsm_NotificationCallScene`
   - `vdsm_NotificationDimChannel`
   - `vdsm_NotificationSetOutputChannelValue`
   - And many more...

5. **Keep-Alive**
   - `vdsm_SendPing` / `vdc_SendPong`

## Your First vDC Implementation

### Prerequisites

- Understanding of your target device protocol (Hue, Sonos, etc.)
- TCP socket programming
- Protocol Buffer library for your language
- Understanding of property trees (see [Property System](./04-property-system.md))

### Implementation Steps

1. **Set up TCP Server**
   ```
   - Listen on port 8440 (default, configurable)
   - Handle incoming connections from vdSM
   - Implement protobuf message encoding/decoding
   ```

2. **Implement Session Lifecycle**
   ```
   - Handle HELLO handshake
   - Store session information
   - Handle BYE for cleanup
   - Implement PING/PONG keep-alive
   ```

3. **Announce Your vDC**
   ```
   - Create unique dSUID for your vDC
   - Send ANNOUNCE_VDC message
   - Populate vDC properties
   ```

4. **Announce Devices**
   ```
   - Discover/connect to external devices
   - Create unique dSUID for each device
   - Send ANNOUNCE_DEVICE for each
   - Build property tree for each device
   ```

5. **Implement Property Access**
   ```
   - Handle GetProperty requests
   - Respond with property values
   - Handle SetProperty requests
   - Send PushNotification for changes
   ```

6. **Implement Notifications**
   ```
   - Handle scene notifications
   - Handle channel/dimming notifications
   - Update external devices accordingly
   ```

7. **Enable Discovery**
   ```
   - Publish Avahi/mDNS service
   - Type: _ds-vdc._tcp
   - Port: 8440 (or your configured port)
   ```

### Minimal Example Flow

```python
# Pseudo-code for minimal vDC host

1. Start TCP server on port 8440

2. On connection from vdSM:
   - Receive HELLO message
   - Send HELLO response with vDC host dSUID
   
3. Announce vDC:
   - Create vDC with unique dSUID
   - Send ANNOUNCE_VDC message
   
4. Announce devices:
   - For each external device:
     - Create vdSD with unique dSUID
     - Send ANNOUNCE_DEVICE message
     
5. Handle requests:
   - On GetProperty: return property values from tree
   - On SetProperty: update device state
   - On Scene notifications: execute scene on device
   - Send PushNotification when device state changes
   
6. Keep connection alive:
   - Respond to PING with PONG
   - Handle BYE for clean shutdown
```

## Next Steps

Now that you understand the basics:

1. **[Core Concepts](./02-core-concepts.md)** - Deeper dive into vDC architecture
2. **[Protocol Basics](./03-protocol-basics.md)** - Detailed protocol information
3. **[Property System](./04-property-system.md)** - **Essential**: Understanding property trees
4. **[Implementation Guide](./10-implementation-guide.md)** - Practical implementation details

## Common Questions

### Q: Do I need to implement all message types?

A: No. Start with the basics:
- Hello handshake
- Announce vDC and devices
- GetProperty/SetProperty
- Basic scene notifications
- Ping/Pong

You can add more features incrementally.

### Q: How do I generate unique dSUIDs?

A: dSUIDs are derived from device identifiers. For virtual devices, you typically use UUID v5 with a namespace. See implementation guide for details.

### Q: What port should I use?

A: The default is 8440, but it's configurable via Avahi/mDNS announcement.

### Q: Do I need to implement discovery?

A: Yes, for automatic integration. Use Avahi/mDNS to advertise your service as `_ds-vdc._tcp`.

### Q: What programming languages are supported?

A: Any language with:
- TCP socket support
- Protocol Buffers support
- Preferably async I/O capabilities

Common choices: C++, Python, Node.js, Go, Java

## Resources

- [Core Concepts](./02-core-concepts.md) - Detailed concepts
- [Property System](./04-property-system.md) - Property tree structure (critical!)
- [Protocol Buffer Reference](./09-protobuf-reference.md) - Complete message definitions
- [Implementation Guide](./10-implementation-guide.md) - Step-by-step implementation

---

[Back to Documentation Index](./README.md) | [Next: Core Concepts →](./02-core-concepts.md)
