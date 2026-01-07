# digitalSTROM Virtual Device Connector (vDC) API Documentation

Welcome to the improved and restructured documentation for the digitalSTROM Virtual Device Connector API. This documentation has been reorganized to provide better accessibility and understanding of the vDC API.

## What is vDC?

The **Virtual Device Connector (vDC)** is a protocol that allows external devices to integrate with the digitalSTROM system. It enables third-party devices to communicate with the digitalSTROM Server (dSS) as if they were native digitalSTROM devices.

## Documentation Structure

This documentation is organized into the following sections:

### 1. [Getting Started](./01-getting-started.md)
- Introduction to vDC concepts
- Key terminology
- Architecture overview
- Quick start guide

### 2. [Core Concepts](./02-core-concepts.md)
- Understanding vDC, vdSM, and vdSD
- Device types and classifications
- Connection management
- Discovery mechanisms

### 3. [Protocol Basics](./03-protocol-basics.md)
- Communication protocol overview
- Message structure (Protocol Buffers)
- Connection lifecycle
- Error handling and response codes

### 4. [Property System](./04-property-system.md)
- **Understanding property trees** (Comprehensive guide)
- Property element structure
- Reading and writing properties
- Property notifications and updates
- Common properties for all entities
- **NEW**: [Property Element Reference](./property-element-reference.md) - Complete reference catalog

### 5. [Virtual Device Connector (vDC)](./05-vdc-reference.md)
- vDC level properties
- vDC capabilities
- vDC announcement and lifecycle

### 6. [Virtual Devices (vdSD)](./06-vdsd-reference.md)
- Device properties and structure
- Device announcement and lifecycle
- Device operations

### 7. [Device Features](./07-device-features.md)
- Inputs (buttons, binary, sensors)
- Outputs and channels
- Scenes
- Actions
- States and properties

### Property Element Reference
- **[Complete Property Element Reference](./property-element-reference.md)**
- Detailed property catalog with interactive navigation
- vDC and device property trees
- All property element descriptions from the original API

### 8. [API Operations](./08-api-operations.md)
- Scene operations (call, save, undo)
- Channel operations (dim, set value)
- Control operations
- Configuration and management

### 9. [Protocol Buffer Messages](./09-protobuf-reference.md)
- Complete message reference from genericVDC.proto
- Request/Response patterns
- Notifications
- Message types and enumerations

### 10. [Implementation Guide](./10-implementation-guide.md)
- Building a vDC host
- Implementing device support
- Best practices
- Common patterns and examples

## Original Documentation

The original PDF documentation has been preserved and converted to markdown format in the `originalDocFiles` directory:

- [vDC-API.md](../originalDocFiles/vDC-API.md) - Core API specification
- [vDC-overview.md](../originalDocFiles/vDC-overview.md) - Overview and introduction
- [vDC-API-properties_JULY_2022.md](../originalDocFiles/vDC-API-properties_JULY_2022.md) - Complete property reference

## Key Improvements in This Documentation

1. **Better Organization**: Information is grouped logically by topic rather than by document structure
2. **Property Tree Focus**: Enhanced explanation of property tree structure and usage
3. **Protocol Buffer Integration**: Clear correlation with the genericVDC.proto implementation
4. **Progressive Learning**: Start with basics and progress to advanced topics
5. **Practical Examples**: More code examples and usage patterns
6. **Cross-References**: Easy navigation between related topics

## Quick Reference

### Essential Terminology

- **vDC (Virtual Device Connector)**: A logical entity representing a class of external devices
- **vDC host**: Network device offering a server socket for vdSM connections
- **vdSM (Virtual digitalSTROM Meter)**: Component that connects vDC hosts to the dS system
- **vdSD (Virtual digitalSTROM Device)**: Individual device in the dS system
- **dSUID**: Unique identifier for devices in the digitalSTROM system
- **Property Tree**: Hierarchical structure for device configuration and state

### Protocol Buffer Message Types

The vDC API uses Protocol Buffers for message encoding. Key message categories:

- **Requests**: Hello, GetProperty, SetProperty, GenericRequest
- **Responses**: Hello, GetProperty, GenericResponse
- **Notifications**: Scene operations, Channel operations, Control updates
- **Device Lifecycle**: Announce, Vanish, Identify
- **Keep-Alive**: Ping/Pong

### Common Property Paths

```
/vdc                    # Virtual Device Connector root
  /name                 # vDC name
  /dSUID                # vDC unique identifier
  /capabilities         # vDC capabilities

/device                 # Virtual Device root
  /name                 # Device name
  /dSUID                # Device unique identifier
  /model                # Device model
  /modelVersion         # Model version
  /inputs               # Device inputs (buttons, sensors)
  /outputs              # Device outputs and channels
  /scenes               # Scene configurations
  /actions              # Available actions
  /states               # Device states
```

## Version Information

This documentation is based on:
- vDC API v1.6-branch (May 4, 2020)
- vDC API Properties (July 5, 2022)
- genericVDC.proto (Protocol Buffer definitions)

## Legal Notice

©2020 digitalSTROM AG. All rights reserved.

This document is intended to assist developers to develop applications that use or integrate digitalSTROM technologies. The digitalSTROM logo is a trademark of digitalSTROM.

---

**Next Steps**: Start with [Getting Started](./01-getting-started.md) to learn the basics of the vDC API.
