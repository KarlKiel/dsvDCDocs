# digitalSTROM Virtual Device Connector Overview

## Introduction

The digitalSTROM Virtual Device Connector (vDC) enables integration of IP-based devices into the digitalSTROM ecosystem, similar to adding physical digitalSTROM terminal blocks (dSD) to the system. IP-based devices in the local network run a vDC to connect to the digitalSTROM System through a virtual digitalSTROM Meter (vdSM).

## Key Concepts

### Terms and Definitions

| Term | Description |
|------|-------------|
| **vDC (Virtual Device Connector)** | A logical entity within the dS system with its own dSUID, representing a type or class of external devices |
| **vDC host** | A network device offering a server socket for vdSM to connect to. One vDC host can host multiple logical vDCs if it supports multiple device classes |
| **vdSM (Virtual digitalSTROM Meter)** | Can connect to one or several vDC hosts to integrate one or several logical vDCs into the dS system |
| **vdSD (Virtual digitalSTROM Device)** | Represents a single device in the dS system and behaves like a real digitalSTROM terminal block (dSD) |
| **vDC session** | A logical connection between a vdSM and a vDC host (representing one or multiple vDCs) |
| **dSUID** | DigitalSTROM Unique Identifier - unique identifier for devices and vDCs |

## Architecture

A vDC can host multiple devices or device classes and is suitable for integrating gateways to other bus or network technologies such as EnOcean, DALI, etc.

### Device Types

There are three different device types that can be distinguished:

#### Device Type 1: Server-Hosted Integration
- Integrates devices through the vdSM and vDC running on the digitalSTROM Server module
- Both vdSM and vDC components run on the digitalSTROM Server
- Suitable for devices that don't have their own processing capability

#### Device Type 2: Device-Hosted Integration
- Integrates devices through the vdSM running on the digitalSTROM Server module
- The vDC component runs directly on the physical device
- Suitable for smart devices with sufficient processing power

#### Device Type 3: Gateway Integration
- Serves as a gateway to integrate multiple devices using bus or network technologies
- Examples: IP, EnOcean, ZigBee, EEBUS, etc.
- Suitable for integrating entire ecosystems of devices

> **Note:** Contact digitalSTROM AG to determine which device type is most suitable for your product.

## Implementation

### Development Effort

The integration effort is reduced to programming the glue logic that connects the vDC API to the real device's API. Two open-source projects are available to abstract the vDC API and provide basic functionality:

#### libdsvdc
- **Type:** Lightweight C library
- **Target:** Device vDCs (Device Types 1 & 2)
- **Use Case:** Best for integrating individual devices or small sets of devices
- **License:** GPL V3 (commercial licenses available from digitalSTROM AG)

#### vdcd
- **Type:** C++ framework
- **Target:** Gateway devices (Device Type 3)
- **Use Case:** Designed for integrating entire device classes with standard dS behavior
- **License:** GPL V3 (commercial licenses available from digitalSTROM AG)

### Platform Support

vDC is available for embedded Linux. Running a vDC on other physical device operating systems is possible but may require additional development effort.

## Discovery

### Service Announcement

vDC hosts announce their services using **Avahi** (the GNU implementation of Apple's Bonjour):

1. vDC hosts announce their presence on the local network
2. vdSMs monitor Avahi announcements to discover available vDC hosts
3. vdSMs connect to at least one discovered vDC host

### Connection Management

- vDC hosts may reject connections if they are already connected to another vdSM
- The vdSM initiating a connection can use an optional whitelist to validate discovered vDC hosts
- Whitelists help prevent wrong connections in complex environments

#### Whitelist Usage

- **Simple installations:** Fully automatic connection without whitelists
- **Complex setups:** Whitelists solve conflicts in showrooms or developer environments
- **Location:** Whitelists are located and maintained on the dSS device

## User Interface

### Configuration UI

- vDCs are displayed in the UI alongside dSM devices
- Each vDC has a virtual "standard room" where all connected vdSDs initially appear
- vdSDs can be assigned to real rooms
- vdSDs behave exactly like physical digitalSTROM terminal blocks

## Licensing

- **Open Source:** vDC code is available under GPL V3
- **Commercial:** Commercial licenses are available from digitalSTROM AG

## Documentation and Resources

### Official Documentation
- vDC API specification
- vDC API properties reference
- This overview document

### Sample Code
Available in:
- libdsvdc library repository
- NetAtmo temperature sensor integration project (reference implementation)

### Access Points
- digitalSTROM developer website
- digitalSTROM alliance website

## Integration Process

1. **Choose Device Type:** Determine which device type (1, 2, or 3) suits your needs
2. **Select Framework:** Choose between libdsvdc (C) or vdcd (C++) based on your device type
3. **Implement Glue Logic:** Connect your device's API to the vDC API
4. **Implement Discovery:** Set up Avahi service announcement
5. **Test Connection:** Verify vdSM can discover and connect to your vDC host
6. **Configure Devices:** Set up vdSDs with appropriate properties and capabilities
7. **Deploy:** Install on target platform and integrate into digitalSTROM installation

## Next Steps

- Review [API Basics](api-basics.md) to understand core API concepts
- Learn about [Connection Management](connection-management.md) for implementing the network protocol
- Study [Session Management](session-management.md) for handling vDC host sessions
- Explore [Device Operations](device-operations.md) for device announcement and control
