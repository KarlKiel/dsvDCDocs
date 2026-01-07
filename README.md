# digitalSTROM vDC API Documentation

This repository contains comprehensive documentation for the digitalSTROM Virtual Device Connector (vDC) API.

## What is this repository?

This repository provides improved, restructured documentation for the vDC API, which enables integration of third-party devices into the digitalSTROM home automation system.

## Documentation

### 📚 Main Documentation

The restructured documentation is in the [`docs/`](./docs/) directory:

**[Start Here: Documentation Index](./docs/README.md)**

### Quick Links

- **[Getting Started](./docs/01-getting-started.md)** - Introduction and quick start
- **[Property System](./docs/04-property-system.md)** - ⭐ Essential guide to property trees
- **[Implementation Guide](./docs/10-implementation-guide.md)** - Step-by-step implementation
- **[Protocol Buffer Reference](./docs/09-protobuf-reference.md)** - Complete message reference

### Documentation Structure

1. **[Getting Started](./docs/01-getting-started.md)** - Introduction, architecture, first steps
2. **[Core Concepts](./docs/02-core-concepts.md)** - Fundamental concepts and terminology
3. **[Protocol Basics](./docs/03-protocol-basics.md)** - Communication protocol details
4. **[Property System](./docs/04-property-system.md)** - ⭐ **Critical**: Property tree structure and usage
5. **[vDC Reference](./docs/05-vdc-reference.md)** - Virtual Device Connector properties
6. **[vdSD Reference](./docs/06-vdsd-reference.md)** - Virtual Device properties
7. **[Device Features](./docs/07-device-features.md)** - Inputs, outputs, scenes, actions
8. **[API Operations](./docs/08-api-operations.md)** - Scene and channel operations
9. **[Protocol Buffer Reference](./docs/09-protobuf-reference.md)** - Complete message catalog
10. **[Implementation Guide](./docs/10-implementation-guide.md)** - Practical implementation guide

## Original Documentation

The original PDF documentation has been converted to markdown and is available in [`originalDocFiles/`](./originalDocFiles/):

- **[vDC-API.md](./originalDocFiles/vDC-API.md)** - Core API specification (converted from PDF)
- **[vDC-overview.md](./originalDocFiles/vDC-overview.md)** - Overview document (converted from PDF)
- **[vDC-API-properties_JULY_2022.md](./originalDocFiles/vDC-API-properties_JULY_2022.md)** - Complete property reference (converted from PDF)
- **[genericVDC.proto](./originalDocFiles/genericVDC.proto)** - Protocol Buffer definitions

Note: The original PDFs are also preserved in this directory for reference.

## What's New in the Restructured Documentation?

The new documentation in [`docs/`](./docs/) provides:

✅ **Better Organization** - Information grouped logically by topic  
✅ **Property Tree Focus** - Comprehensive guide to property trees (the core of vDC API)  
✅ **Protocol Buffer Integration** - Clear correlation with genericVDC.proto  
✅ **Progressive Learning** - Start simple, progress to advanced topics  
✅ **Practical Examples** - Real code examples and patterns  
✅ **Easy Navigation** - Cross-references and clear structure  

## Key Highlights

### Property System

The **[Property System guide](./docs/04-property-system.md)** is essential reading. It thoroughly explains:

- What property trees are and why they matter
- PropertyElement structure from the protobuf spec
- How to query and navigate property trees
- Reading and writing properties
- Common property paths
- Implementation best practices

This was identified as a key area needing better documentation and has been extensively covered.

### Implementation Guide

The **[Implementation Guide](./docs/10-implementation-guide.md)** provides:

- Complete architecture design
- Step-by-step implementation
- Working Python code examples
- Best practices
- Testing strategies

### Protocol Buffer Reference

The **[Protocol Buffer Reference](./docs/09-protobuf-reference.md)** catalogs:

- All message types from genericVDC.proto
- Field descriptions and usage
- Request/response patterns
- Error codes and handling

## Quick Start

1. Read **[Getting Started](./docs/01-getting-started.md)** for an overview
2. Understand **[Property System](./docs/04-property-system.md)** - this is critical!
3. Review **[Protocol Basics](./docs/03-protocol-basics.md)** for communication details
4. Follow the **[Implementation Guide](./docs/10-implementation-guide.md)** for practical steps

## About the vDC API

The Virtual Device Connector API allows external devices to integrate with digitalSTROM:

- **Protocol**: Protocol Buffers over TCP
- **Port**: Default 8440
- **Discovery**: Avahi/mDNS
- **Languages**: Any with protobuf support

## Version Information

Documentation based on:
- vDC API v1.6-branch (May 4, 2020)
- vDC API Properties (July 5, 2022)
- genericVDC.proto (Protocol Buffer definitions)

## Documentation Status

**Phase 1: PDF Conversion** ✅ Complete
- Converted all three vDC PDF documents to markdown (1:1 conversion)
- Files preserved in `originalDocFiles/` directory

**Phase 2: Restructured Documentation** ✅ Complete
- Created comprehensive new documentation structure
- Special focus on property tree system
- Added practical implementation guide
- Integrated Protocol Buffer specifications
- 130KB+ of new structured documentation

## Contributing

This documentation was created to make the vDC API more accessible and easier to understand. The restructured documentation maintains all information from the original PDFs while improving organization and clarity.

## Legal

©2020 digitalSTROM AG. All rights reserved.

The digitalSTROM logo is a trademark of digitalSTROM. This documentation is intended to assist developers in creating applications that integrate with digitalSTROM technologies.

## Questions?

Start with the **[Documentation Index](./docs/README.md)** for a complete overview and navigation guide.

---

**Ready to get started?** → **[Begin with Getting Started](./docs/01-getting-started.md)**
