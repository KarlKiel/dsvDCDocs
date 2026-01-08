# Getting Started with vDC API

## Introduction

The digitalSTROM Virtual Device Connector (vDC) API enables integration of third-party devices into the digitalSTROM ecosystem. This guide will help you understand the basics and get started with implementing a vDC host.

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
| **Property Element** | - | Key/Value pair to contain single configuration, state or capability or Key/Elements pair to contain collection of other Property Elements defining the Key | 

## Architecture Overview

### Components and Relationships

1. **vdSM** (integrated Part of dSS)
   - Discovers vDC hosts (via Avahi/mDNS)
   - Initiates TCP connection to vDC host
   - Manages communication protocol
   - Routes messages between dSS and vDC host
     
2. **vDC Host** (Your Server Implementation)
   - Runs as a network service
   - Listens on a TCP port (default: 8440)
   - Can host multiple vDCs
   - Manages connection to external devices (Hue, Sonos, etc.)
   - Routes messages between vDC host and dss

4. **vDC** (Your Implementation running on the vDC Host)
   - Can manage multiple vdSDs (devices)
   - Handles communication logic between vdSM and vdSDs
   - Encodes and decodes messages for communication between vdSM and vdSDs

5. **vdSD** (Individual Device)
   - Represents one external device as dS device
   - Responds to actions and notifications
   - Can actively push information
   - a physical device can be represented by several individual vdSDs

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

[Back to Documentation Index](./README.md) | [Next: Core Concepts →](./02-core-concepts.md)
