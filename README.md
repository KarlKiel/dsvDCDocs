# digitalSTROM Virtual Device Connector (vDC) API Documentation

This repository contains comprehensive documentation for the digitalSTROM Virtual Device Connector (vDC) API, which enables integration of external devices into the digitalSTROM ecosystem.

## Overview

The vDC-API allows developers to create virtual device connectors that bridge non-digitalSTROM devices into the digitalSTROM system. This enables seamless control and automation of third-party devices using the digitalSTROM platform.

## Documentation Structure

### Getting Started
- [Quick Start Guide](docs/quick-start.md) - Get up and running quickly with code examples
- [Overview](docs/overview.md) - Introduction to vDC concepts and architecture
- [API Basics](docs/api-basics.md) - Core concepts and conventions

### Core Documentation
- [Connection Management](docs/connection-management.md) - TCP connections and Protocol Buffer transport
- [Session Management](docs/session-management.md) - vDC host session lifecycle
- [Device Operations](docs/device-operations.md) - Device announcement and operation
- [Properties](docs/properties.md) - Property system and available properties
- [Actions and Notifications](docs/actions-notifications.md) - Available actions and notification messages
- [Error Handling](docs/error-handling.md) - Error codes and error handling

### Reference
- [Protocol Reference](docs/protocol-reference.md) - Complete Protocol Buffer message reference

## Key Concepts

### Terms

| Term | Description |
|------|-------------|
| **vDC** | Virtual device connector - A logical entity representing a type or class of external devices |
| **vDC host** | Network device offering a server socket for vdSM to connect to |
| **vdSM** | Virtual digitalSTROM Meter - Connects to vDC hosts to integrate devices into the dS system |
| **vdSD** | Virtual digitalSTROM device - Represents a single device in the dS system |
| **dSUID** | DigitalSTROM Unique Identifier - Unique identifier for devices and vDCs |

## Quick Start

**New to vDC-API?** Start with the [Quick Start Guide](docs/quick-start.md) for a hands-on introduction with code examples.

**Detailed Learning Path:**

1. **Understand the Architecture**: Read the [Overview](docs/overview.md) to understand how vDCs fit into the digitalSTROM ecosystem
2. **Learn the Protocol**: Review [Connection Management](docs/connection-management.md) to understand the communication protocol
3. **Implement a vDC Host**: Follow [Session Management](docs/session-management.md) to implement the connection lifecycle
4. **Add Devices**: Use [Device Operations](docs/device-operations.md) to announce and manage devices
5. **Handle Properties**: Implement property access as described in [Properties](docs/properties.md)

## API Versions

This documentation covers vDC-API versions up to v3, with notes on version-specific features where applicable.

## Original Documentation

The original PDF documentation files are preserved in the `originalDocFiles/` directory:
- `vDC-overview.pdf` - Original overview document
- `vDC-API.pdf` - Original API specification
- `vDC-API-properties_ JULY 2022.pdf` - Original properties documentation
- `genericVDC.proto` - Protocol Buffer schema definition

## License

See the [LICENSE](LICENSE) file for details.

## Support

For questions and support regarding the digitalSTROM vDC-API, please refer to the official digitalSTROM documentation and support channels.

---

*Note: This documentation is a community-driven rewrite of the official digitalSTROM vDC-API documentation for improved accessibility and usability.*
