# Property Element Reference

## Overview

This document provides a comprehensive reference for all property elements in the digitalSTROM Virtual Device Connector (vDC) API. It serves as a detailed catalog of properties available at both vDC and vdSD (device) levels.

**Purpose**: Use this reference to understand the exact structure, type, access mode, and meaning of each property element in the property tree.

**Organization**: Properties are organized following the property tree structure:
- **vDC Tree**: Common properties and vDC-specific properties
- **Device (vdSD) Tree**: Common properties, device-specific properties, and input/output properties

**Notation**:
- **(R)** = Required property (must be implemented)
- **(O)** = Optional property (may or may not be present)
- **(R/W)** = Read/Write property
- **[propertyName]** = Container or array property

---

## Table of Contents

- [vDC Property Tree](#vdc-property-tree)
  - [Common Properties (vDC Level)](#common-properties-vdc-level)
  - [vDC-Specific Properties](#vdc-specific-properties)
- [Device (vdSD) Property Tree](#device-vdsd-property-tree)
  - [Common Properties (Device Level)](#common-properties-device-level)
  - [Device-Specific Properties](#device-specific-properties)
  - [Button Input Properties](#button-input-properties)
  - [Binary Input Properties](#binary-input-properties)
  - [Sensor Input Properties](#sensor-input-properties)
  - [Output Properties](#output-properties)
  - [Channel Properties](#channel-properties)
  - [Scene Properties](#scene-properties)
  - [Action Properties](#action-properties)
  - [State and Property Descriptions](#state-and-property-descriptions)
  - [Event Properties](#event-properties)

---

## vDC Property Tree

The following tree shows all properties available when querying a vDC entity (using the vDC's dSUID).

### Interactive Property Tree - vDC Level

```
/ (root - all properties belong to the vDC)
├── dSUID (R) - string, 34 hex characters
├── displayId (O) - string
├── type (R) - string, value: "vDC"
├── model (R) - string
├── modelVersion (O) - string
├── modelUID (R) - string
├── hardwareVersion (O) - string
├── hardwareGuid (O) - string, URN format
├── hardwareModelGuid (O) - string, URN format
├── vendorName (O) - string
├── vendorGuid (O) - string, URN format
├── oemGuid (O) - string
├── oemModelGuid (O) - string
├── configURL (O) - string
├── deviceIcon16 (O) - binary (16x16 PNG)
├── deviceIconName (O) - string
├── name (R) - string, read/write
├── deviceClass (O) - string
├── deviceClassVersion (O) - string
├── active (O) - boolean
├── implementationId (R) - string
├── zoneID (R/W) - uint64
└── [capabilities] (R) - container
    ├── metering (O) - boolean
    ├── identification (O) - boolean
    └── dynamicDefinitions (O) - boolean
```

**Interactive Links**:
- [dSUID](#dsuid)
- [displayId](#displayid)
- [type](#type)
- [model](#model)
- [modelVersion](#modelversion)
- [modelUID](#modeluid)
- [hardwareVersion](#hardwareversion)
- [hardwareGuid](#hardwareguid)
- [hardwareModelGuid](#hardwaremodelguid)
- [vendorName](#vendorname)
- [vendorGuid](#vendorguid)
- [oemGuid](#oemguid)
- [oemModelGuid](#oemmodelguid)
- [configURL](#configurl)
- [deviceIcon16](#deviceicon16)
- [deviceIconName](#deviceiconname)
- [name](#name)
- [deviceClass](#deviceclass)
- [deviceClassVersion](#deviceclassversion)
- [active](#active)
- [implementationId](#implementationid)
- [zoneID (vDC)](#zoneid-vdc)
- [capabilities](#capabilities)
  - [metering](#metering)
  - [identification](#identification)
  - [dynamicDefinitions](#dynamicdefinitions)

---

## Common Properties (vDC Level)

These properties are common to all addressable entities. They must be supported by entities which can be addressed via a dSUID using this API.

### dSUID

**Access**: Read  
**Type**: string (34 hex characters, 2*17)  
**Required**: Yes  

**Description**: The dSUID of the entity. Normally not used in regular vDC API calls, as these require the dSUID in the getProperty() call. Useful for debugging.

---

### displayId

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Human-readable identification usually printed on the physical device to identify the device, if available.

---

### type

**Access**: Read  
**Type**: string  
**Required**: Yes  

**Description**: Type of entity.

**Valid Values**:
- `"vdSD"` : virtual dS device
- `"vDC"` : a logical vDC
- `"vDChost"` : the vDC host
- `"vdSM"` : a vdSM

For vDC entities, this value is always `"vDC"`.

---

### model

**Access**: Read  
**Type**: string  
**Required**: Yes  

**Description**: Human-readable model string of the entity. Should be descriptive enough to allow a human to associate it with a kind of hardware or software. Is mapped to "hardwareInfo" in vdsm and upstream.

---

### modelVersion

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Human-readable model version string of the device, if available.

---

### modelUID

**Access**: Read  
**Type**: string  
**Required**: Yes  

**Description**: digitalSTROM system unique ID for the functional model of the entity.

**Important Notes**:
- modelUID must be equal between all functionally identical entities (especially devices) in the dS system.
- If different connected hardware devices provide EXACTLY the same dS functionality, these devices MAY have the same modelUID but will have different hardwareModelGuid.
- Vice versa, for example two identical hardware input devices will have the same hardwareModelGuid, but different modelUID if one input is mapped as a button, and the other as a binaryInput.

---

### hardwareVersion

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Human-readable string describing the hardware device's version represented by this entity, if available.

---

### hardwareGuid

**Access**: Read  
**Type**: string (URN-like format)  
**Required**: No (Optional)  

**Description**: Hardware's native globally unique identifier (GUID), if any, in URN-like format: `formatname:actualID`

**The following formats are in use**:
- `gs1:(01)ggggg(21)sssss` = GS.1 formatted GTIN plus serial number
- `macaddress:MMMMM` = MAC Address
- `enoceanaddress:XXXXXXXX` = 8 hex digits EnOcean device address
- `uuid:UUUUUUU` = UUID

---

### hardwareModelGuid

**Access**: Read  
**Type**: string (URN-like format)  
**Required**: No (Optional)  

**Description**: Hardware model's native globally unique identification, if any, in URN-like format: `formatname:actualID`

**The following formats are in use**:
- `gs1:(01)ggggg` = GS.1 formatted GTIN
- `enoceaneep:oofftt` = 6 hex digits EnOcean Equipment Profile (EEP) number

---

### vendorName

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Human-readable string of the device manufacturer or vendor.

---

### vendorGuid

**Access**: Read  
**Type**: string (URN-like format)  
**Required**: No (Optional)  

**Description**: Globally unique identification of the vendor, in URN-like format.

**The following formats are in use**:
- `enoceanvendor:vvv[:name]` = 3 hex digits EnOcean vendor ID, optionally followed by a colon and the clear text vendor name if known
- `vendorname:name` = clear text name of the vendor
- `gs1:(412)lllll` = GS1 formatted Global Location Number of the vendor

---

### oemGuid

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Globally unique identifier (GUID) of the product the hardware is embedded in, if any - see hardwareGuid for format variants.

---

### oemModelGuid

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Globally unique identifier (GUID) of the product model (if any) in `gs1:(01)ggggggggggggg` format. Often referred to as GTIN.

---

### configURL

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Full URL how to reach the web configuration of this device (if any).

---

### deviceIcon16

**Access**: Read  
**Type**: binary (PNG image)  
**Required**: No (Optional)  

**Description**: 16x16 pixel PNG image to represent this device in the digitalSTROM configurator UI.

---

### deviceIconName

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Filename-safe name for the icon (a-z, 0-9, _, -, no spaces or funny characters!). This allows for more efficient caching in Web UIs - many devices might have the same icon, so web UIs don't need to load the actual data (deviceIcon16) for every device again, as long as devices show the same deviceIconName.

---

### name

**Access**: Read/Write  
**Type**: string  
**Required**: Yes  

**Description**: User-specified name of the entity. Is also stored upstreams in the vdSM and further up, but is useful for the vDC to know for configuration and debugging.

The vDC usually generates descriptive default names for newly connected devices. When the vdSM registers a device, it should read this property and propagate the name towards the dSS. When the user changes the name via the dSS configurator, this property should be updated with the new name.

---

### deviceClass

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: digitalSTROM defined unique name of a device class profile.

---

### deviceClassVersion

**Access**: Read  
**Type**: string  
**Required**: No (Optional)  

**Description**: Revision number of the device class profile.

---

### active

**Access**: Read  
**Type**: boolean  
**Required**: No (Optional)  

**Description**: Operation state of the addressable entity. If this is not true, this means the associated hardware cannot operate normally at this time. This might be due to communication problems, radio range limitations, missing configuration etc.

When a vDC detects a change in active, this will be reported to vDC API clients via pushProperty. However, vDCs cannot guarantee timely updates of active in all cases. In particular, detecting hardware becoming disconnected or no longer reachable might involve long timeouts or might not be feasible at all, depending on the hardware type.

**Note**: active is optional only to maintain backwards compatibility with earlier versions of this specification. Still, active should be implemented in all modern vDCs.

**Note**: active does not replace the ping/pong mechanism that always could be used to poll the operation state. ping/pong must be implemented in all vDCs.

---

## vDC-Specific Properties

These properties apply specifically to vDC entities (entities with `type` = "vDC").

### implementationId

**Access**: Read  
**Type**: string  
**Required**: Yes  

**Description**: Unique id used to identify vdc implementation. Non-digitalSTROM vdcs must use "x-company-" prefix to avoid collisions.

---

### zoneID (vDC)

**Access**: Read/Write  
**Type**: integer (global dS Zone ID)  
**Required**: Yes  

**Description**: This should be updated by the vdSM to reflect the default zone the vdc has.

---

### capabilities

**Access**: Read  
**Type**: list of property elements (container)  
**Required**: Yes  

**Description**: Descriptions (invariable properties) of the vdc's capabilities. Each capability is represented as a property element (key/value).

**Child Properties**: See below for individual capability properties.

---

#### metering

**Access**: Read  
**Type**: boolean  
**Required**: No (Optional)  
**Parent**: capabilities  

**Description**: If true, the vdc provides metering data.

---

#### identification

**Access**: Read  
**Type**: boolean  
**Required**: No (Optional)  
**Parent**: capabilities  

**Description**: If true, the vdc provides a way of identifying itself. E.g. a LED that can be blinked.

---

#### dynamicDefinitions

**Access**: Read  
**Type**: boolean  
**Required**: No (Optional)  
**Parent**: capabilities  

**Description**: If true, the vdc supports dynamic device definitions, e.g. propertyDescriptions and actionsDescriptions.


---

## Device (vdSD) Property Tree

The following tree shows all properties available when querying a device entity (using the device's dSUID).

### Interactive Property Tree - Device Level

```
/ (root - all properties belong to the device)
├── Common Properties (same as vDC, Section 2 of original API)
│   ├── dSUID (R) - string, 34 hex characters
│   ├── displayId (O) - string
│   ├── type (R) - string, value: "vdSD"
│   ├── model (R) - string
│   ├── modelVersion (O) - string
│   ├── modelUID (R) - string
│   ├── hardwareVersion (O) - string
│   ├── hardwareGuid (O) - string, URN format
│   ├── hardwareModelGuid (O) - string, URN format
│   ├── vendorName (O) - string
│   ├── vendorGuid (O) - string, URN format
│   ├── oemGuid (O) - string
│   ├── oemModelGuid (O) - string
│   ├── configURL (O) - string
│   ├── deviceIcon16 (O) - binary (16x16 PNG)
│   ├── deviceIconName (O) - string
│   ├── name (R) - string, read/write
│   ├── deviceClass (O) - string
│   ├── deviceClassVersion (O) - string
│   └── active (O) - boolean
│
├── Device-Specific Properties (Section 4.1.1)
│   ├── primaryGroup (R) - uint64 (color/function)
│   └── zoneID (R/W) - uint64
│
├── Button Input Properties (Section 4.2)
│   ├── [buttonInputDescriptions] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       ├── name (R) - string
│   │       ├── dsIndex (R) - uint64 (0..N-1)
│   │       ├── supportsLocalKeyMode (R) - boolean
│   │       ├── buttonID (O) - uint64
│   │       ├── buttonType (R) - uint64
│   │       └── buttonElementID (R) - uint64
│   │
│   ├── [buttonInputSettings] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       ├── group (R/W) - uint64
│   │       ├── function (R/W) - uint64
│   │       ├── mode (R/W) - uint64
│   │       ├── channel (R/W) - uint64
│   │       ├── setsLocalPriority (R/W) - boolean
│   │       └── callsPresent (R/W) - boolean
│   │
│   └── [buttonInputStates] (O) - array
│       └── [0], [1], ... [N-1]
│           ├── value (R) - boolean or NULL
│           ├── clickType (R) - uint64
│           ├── age (R) - double or NULL
│           ├── error (R) - uint64
│           ├── actionId (O) - uint64 (alternative to value/clickType)
│           └── actionMode (O) - uint64 (alternative to value/clickType)
│
├── Binary Input Properties (Section 4.3)
│   ├── [binaryInputDescriptions] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       ├── name (R) - string
│   │       ├── dsIndex (R) - uint64 (0..N-1)
│   │       ├── inputType (R) - uint64
│   │       └── inputUsage (R) - uint64
│   │
│   ├── [binaryInputSettings] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       ├── group (R/W) - uint64
│   │       ├── minPushInterval (R/W) - uint64
│   │       └── changesOnlyInterval (R/W) - uint64
│   │
│   └── [binaryInputStates] (O) - array
│       └── [0], [1], ... [N-1]
│           ├── value (R) - boolean or NULL
│           ├── extendedValue (R) - uint64 or NULL
│           ├── age (R) - double or NULL
│           └── error (R) - uint64
│
├── Sensor Input Properties (Section 4.4)
│   ├── [sensorDescriptions] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       ├── name (R) - string
│   │       ├── dsIndex (R) - uint64 (0..N-1)
│   │       ├── sensorType (R) - uint64
│   │       ├── sensorUsage (R) - uint64
│   │       ├── min (R) - double
│   │       ├── max (R) - double
│   │       ├── resolution (R) - double
│   │       ├── updateInterval (R) - double
│   │       └── aliveSignInterval (R) - double
│   │
│   ├── [sensorSettings] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       └── group (R/W) - uint64
│   │
│   └── [sensorStates] (O) - array
│       └── [0], [1], ... [N-1]
│           ├── value (R) - double or NULL
│           ├── age (R) - double or NULL
│           └── error (R) - uint64
│
├── Output Properties (Section 4.1.3, 4.8, 4.9)
│   ├── [outputDescription] (O) - container, for devices with output
│   │   ├── function (R) - string
│   │   └── outputUsage (R) - uint64
│   │
│   ├── [outputSettings] (O) - container
│   │   ├── mode (R/W) - uint64
│   │   └── ... (device-specific settings)
│   │
│   ├── [outputState] (O) - container
│   │   └── ... (device-specific state info)
│   │
│   ├── [channelDescriptions] (O) - array, for devices with output
│   │   └── [0], [1], ... [N-1]
│   │       ├── name (R) - string
│   │       ├── channelType (R) - uint64
│   │       ├── dsIndex (R) - uint64 (index 0 is default channel)
│   │       ├── min (R) - double
│   │       ├── max (R) - double
│   │       └── resolution (R) - double
│   │
│   ├── [channelSettings] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       └── (currently no standard settings defined)
│   │
│   └── [channelStates] (O) - array
│       └── [0], [1], ... [N-1]
│           ├── value (R) - double (read-only, use setOutputChannelValue)
│           └── age (R) - double or NULL
│
├── Scene Properties (Section 4.10)
│   └── [scenes] (O) - array, for devices with output
│       └── [0], [5], [17], ... (named by scene number)
│           └── [channels] (R/W) - array
│               └── [0], [1], ... [N-1] (by channel index)
│                   ├── value (R/W) - double
│                   ├── dontCare (R/W) - boolean
│                   └── automatic (R/W) - boolean
│
├── Action Properties (Section 4.5)
│   ├── [deviceActionDescriptions] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       └── ... (action description properties)
│   │
│   └── [customActions] (O) - array
│       └── [0], [1], ... [N-1]
│           └── ... (custom action properties, read/write)
│
├── State Properties (Section 4.6)
│   ├── [deviceStateDescriptions] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       └── ... (state description properties)
│   │
│   └── [deviceStates] (O) - array
│       └── [0], [1], ... [N-1]
│           └── ... (state values, read/write)
│
├── Device Property Descriptions (Section 4.6)
│   ├── [devicePropertyDescriptions] (O) - array
│   │   └── [0], [1], ... [N-1]
│   │       └── ... (property descriptions)
│   │
│   └── [deviceProperties] (O) - array
│       └── [0], [1], ... [N-1]
│           └── ... (property values, read/write)
│
└── Event Properties (Section 4.7)
    └── [deviceEventDescriptions] (O) - array
        └── [0], [1], ... [N-1]
            └── ... (event descriptions)
```

**Interactive Links - Common Properties**:
- [dSUID](#dsuid) - [displayId](#displayid) - [type](#type) - [model](#model) - [modelVersion](#modelversion)
- [modelUID](#modeluid) - [hardwareVersion](#hardwareversion) - [hardwareGuid](#hardwareguid)
- [hardwareModelGuid](#hardwaremodelguid) - [vendorName](#vendorname) - [vendorGuid](#vendorguid)
- [oemGuid](#oemguid) - [oemModelGuid](#oemmodelguid) - [configURL](#configurl)
- [deviceIcon16](#deviceicon16) - [deviceIconName](#deviceiconname) - [name](#name)
- [deviceClass](#deviceclass) - [deviceClassVersion](#deviceclassversion) - [active](#active)

**Interactive Links - Device-Specific**:
- [primaryGroup](#primarygroup) - [zoneID (Device)](#zoneid-device)

**Interactive Links - Buttons**:
- [buttonInputDescriptions](#buttoninputdescriptions): [name](#name-button) - [dsIndex](#dsindex-button) - [supportsLocalKeyMode](#supportslocalke ymode) - [buttonID](#buttonid) - [buttonType](#buttontype) - [buttonElementID](#buttonelementid)
- [buttonInputSettings](#buttoninputsettings): [group](#group-button-settings) - [function](#function-button-settings) - [mode](#mode-button-settings) - [channel](#channel-button-settings) - [setsLocalPriority](#setslocalpriority) - [callsPresent](#callspresent)
- [buttonInputStates](#buttoninputstates): [value](#value-button-state) - [clickType](#clicktype) - [age](#age-button-state) - [error](#error-button-state) - [actionId](#actionid) - [actionMode](#actionmode)

**Interactive Links - Binary Inputs**:
- [binaryInputDescriptions](#binaryinputdescriptions): [name](#name-binary) - [dsIndex](#dsindex-binary) - [inputType](#inputtype) - [inputUsage](#inputusage)
- [binaryInputSettings](#binaryinputsettings): [group](#group-binary-settings) - [minPushInterval](#minpushinterval) - [changesOnlyInterval](#changesonlyinterval)
- [binaryInputStates](#binaryinputstates): [value](#value-binary-state) - [extendedValue](#extendedvalue) - [age](#age-binary-state) - [error](#error-binary-state)

**Interactive Links - Sensors**:
- [sensorDescriptions](#sensordescriptions): [name](#name-sensor) - [dsIndex](#dsindex-sensor) - [sensorType](#sensortype) - [sensorUsage](#sensorusage) - [min](#min-sensor) - [max](#max-sensor) - [resolution](#resolution-sensor) - [updateInterval](#updateinterval-sensor) - [aliveSignInterval](#alivesigninterval)
- [sensorSettings](#sensorsettings): [group](#group-sensor-settings)
- [sensorStates](#sensorstates): [value](#value-sensor-state) - [age](#age-sensor-state) - [error](#error-sensor-state)

**Interactive Links - Outputs & Channels**:
- [outputDescription](#outputdescription): [function](#function-output) - [outputUsage](#outputusage)
- [outputSettings](#outputsettings): [mode](#mode-output-settings)
- [outputState](#outputstate)
- [channelDescriptions](#channeldescriptions): [name](#name-channel) - [channelType](#channeltype) - [dsIndex](#dsindex-channel) - [min](#min-channel) - [max](#max-channel) - [resolution](#resolution-channel)
- [channelSettings](#channelsettings) - [channelStates](#channelstates): [value](#value-channel-state) - [age](#age-channel-state)

**Interactive Links - Scenes**:
- [scenes](#scenes) - [channels (Scene)](#channels-scene) - [value (Scene)](#value-scene-channel) - [dontCare](#dontcare) - [automatic](#automatic)

**Interactive Links - Other**:
- [deviceActionDescriptions](#deviceactiondescriptions) - [customActions](#customactions)
- [deviceStateDescriptions](#devicestatedescriptions) - [deviceStates](#devicestates)
- [devicePropertyDescriptions](#devicepropertydescriptions) - [deviceProperties](#deviceproperties)
- [deviceEventDescriptions](#deviceeventdescriptions)

---

## Common Properties (Device Level)

Device entities (vdSD) have the same common properties as vDC entities, with the key difference that the `type` property value is "vdSD" instead of "vDC". See the [Common Properties (vDC Level)](#common-properties-vdc-level) section for detailed descriptions of these properties.

---

## Device-Specific Properties

These properties apply specifically to device (vdSD) entities.

### primaryGroup

**Access**: Read  
**Type**: integer (dS class number)  
**Required**: Yes  

**Description**: Basic class (color) of the device.

---

### zoneID (Device)

**Access**: Read/Write  
**Type**: integer (global dS Zone ID)  
**Required**: Yes  

**Description**: This should be updated by the vdSM to reflect the zone the device is in. The vDC may use this value to optimize zone calls (i.e. bundle calls to actual hardware if single device calls are slow).

---

## Button Input Properties

Virtual devices can have zero to several buttons. The following properties provide access to button input descriptions, settings, and states.

### buttonInputDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions (invariable properties) of the buttons in the device. Represented as a list of property elements, one for each button in the device. The property elements are named sequentially "0", "1", etc.

**Array Element Properties**: See below for individual button description properties.

---

#### name (Button)

**Access**: Read  
**Type**: string  
**Required**: Yes  
**Parent**: buttonInputDescriptions[N]  

**Description**: Human readable name/number for the input (e.g. matching labels for hardware connectors).

---

#### dsIndex (Button)

**Access**: Read  
**Type**: integer (0..N-1)  
**Required**: Yes  
**Parent**: buttonInputDescriptions[N]  

**Description**: Index number where N=number of buttons in this device.

---

#### supportsLocalKeyMode

**Access**: Read  
**Type**: boolean  
**Required**: Yes  
**Parent**: buttonInputDescriptions[N]  

**Description**: Can be local button.

---

#### buttonID

**Access**: Read  
**Type**: integer (0..n)  
**Required**: No (Optional)  
**Parent**: buttonInputDescriptions[N]  

**Description**: ID of physical button. No ID means no fixed assignment to a button. All elements of a multi-function hardware button must have the same buttonID.

---

#### buttonType

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: buttonInputDescriptions[N]  

**Description**: Type of physical button.

**Valid Values**:
- `0`: undefined
- `1`: single pushbutton
- `2`: 2-way pushbutton
- `3`: 4-way navigation button
- `4`: 4-way navigation with center button
- `5`: 8-way navigation with center button
- `6`: on-off switch

---

#### buttonElementID

**Access**: Read  
**Type**: integer  
**Required**: Yes  
**Parent**: buttonInputDescriptions[N]  

**Description**: Element of multi-contact button.

**Valid Values**:
- `0`: center
- `1`: down
- `2`: up
- `3`: left
- `4`: right
- `5`: upper left
- `6`: lower left
- `7`: upper right
- `8`: lower right

**Note**: For undefined buttonType, buttonElement just enumerates the elements (0..numElements-1).

---

### buttonInputSettings

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Settings (properties that can be changed and are stored persistently in the vDC) of the buttons in the device.

**Array Element Properties**: See below for individual button setting properties.

---

#### group (Button Settings)

**Access**: Read/Write  
**Type**: integer  
**Required**: Yes  
**Parent**: buttonInputSettings[N]  

**Description**: dS group number.

---

#### function (Button Settings)

**Access**: Read/Write  
**Type**: integer (0..15)  
**Required**: Yes  
**Parent**: buttonInputSettings[N]  

**Description**: See LTNUM descriptions (0: device, 5: room, ...).

---

#### mode (Button Settings)

**Access**: Read/Write  
**Type**: integer  
**Required**: Yes  
**Parent**: buttonInputSettings[N]  

**Description**: Button mode.

**Valid Values**:
- `255`: inactive
- `0`: standard
- `2`: presence
- `5..8`: button1..4 down
- `9..12`: button1..4 up

---

#### channel (Button Settings)

**Access**: Read/Write  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: buttonInputSettings[N]  

**Description**: Channel this button should control.

**Valid Values**:
- `0`: (default) button controls the default channel
- `1..191`: reserved for digitalSTROM standard channel types
- `192..239`: device specific channel types

---

#### setsLocalPriority

**Access**: Read/Write  
**Type**: boolean  
**Required**: Yes  
**Parent**: buttonInputSettings[N]  

**Description**: Button should set local priority.

---

#### callsPresent

**Access**: Read/Write  
**Type**: boolean  
**Required**: Yes  
**Parent**: buttonInputSettings[N]  

**Description**: Button should call present (if system state is absent).

---

### buttonInputStates

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: State (changing during operation of the device, but not persistently stored in the vDC) of a button.

**Array Element Properties**: See below for individual button state properties.

---

#### value (Button State)

**Access**: Read  
**Type**: boolean or NULL  
**Required**: Yes  
**Parent**: buttonInputStates[N]  

**Description**: Current button state.

**Valid Values**:
- `false`: inactive
- `true`: active
- `NULL`: unknown state

---

#### clickType

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: buttonInputStates[N]  

**Description**: Most recent click state of the button.

**Valid Values**:
- `0`: tip_1x
- `1`: tip_2x
- `2`: tip_3x
- `3`: tip_4x
- `4`: hold_start
- `5`: hold_repeat
- `6`: hold_end
- `7`: click_1x
- `8`: click_2x
- `9`: click_3x
- `10`: short_long
- `11`: local_off
- `12`: local_on
- `13`: short_short_long
- `14`: local_stop
- `255`: idle (no recent click)

---

#### age (Button State)

**Access**: Read  
**Type**: double or NULL  
**Required**: Yes  
**Parent**: buttonInputStates[N]  

**Description**: Age of the state shown in the value and clickType fields in seconds. If no recent state is known, returns NULL.

---

#### error (Button State)

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: buttonInputStates[N]  

**Description**: Error state of the button.

**Valid Values**:
- `0`: ok
- `1`: open circuit
- `2`: short circuit
- `4`: bus connection problem
- `5`: low battery in device
- `6`: other device error

---

#### actionId

**Access**: Read  
**Type**: integer  
**Required**: No (Optional)  
**Parent**: buttonInputStates[N]  

**Description**: Scene id. Alternatively, buttons can emit direct scene calls instead of button clicks. The buttonInputState can contain the actionId and actionMode properties instead of value and clickType when the most current button action was not a regular click event, but a direct scene call.

---

#### actionMode

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: No (Optional)  
**Parent**: buttonInputStates[N]  

**Description**: Action mode for direct scene calls.

**Valid Values**:
- `0`: normal
- `1`: force
- `2`: undo


---

## Binary Input Properties

Virtual devices can have zero to several binary (digital) inputs. The following properties provide access to binary input descriptions, settings, and states.

### binaryInputDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions of the binary inputs in the device.

**Array Element Properties**: See below for individual binary input description properties.

---

#### name (Binary)

**Access**: Read  
**Type**: string  
**Required**: Yes  
**Parent**: binaryInputDescriptions[N]  

**Description**: Human readable name/number for the input (e.g. matching labels for hardware connectors).

---

#### dsIndex (Binary)

**Access**: Read  
**Type**: integer (0..N-1)  
**Required**: Yes  
**Parent**: binaryInputDescriptions[N]  

**Description**: Index number where N=number of binary inputs in this device.

---

#### inputType

**Access**: Read  
**Type**: integer  
**Required**: Yes  
**Parent**: binaryInputDescriptions[N]  

**Description**: Input type (inputs with binary functions supported only).

**Valid Values**:
- `0`: poll only
- `1`: detects changes

---

#### inputUsage

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: binaryInputDescriptions[N]  

**Description**: Describes the usage field for the input (beyond device color).

**Valid Values**:
- `0`: undefined (generic usage or unknown)
- `1`: room climate
- `2`: outdoor climate
- `3`: climate setting (from user)

---

### binaryInputSettings

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Settings (properties that can be changed and are stored persistently in the vDC) of the binary inputs in the device.

**Array Element Properties**: See below for individual binary input setting properties.

---

#### group (Binary Settings)

**Access**: Read/Write  
**Type**: integer  
**Required**: Yes  
**Parent**: binaryInputSettings[N]  

**Description**: dS group number.

---

#### minPushInterval

**Access**: Read/Write  
**Type**: uint64  
**Required**: Yes  
**Parent**: binaryInputSettings[N]  

**Description**: Minimum interval between pushes of changed state in seconds.

---

#### changesOnlyInterval

**Access**: Read/Write  
**Type**: uint64  
**Required**: Yes  
**Parent**: binaryInputSettings[N]  

**Description**: Minimum interval between pushes with same value (in case sensor hardware sends update, but with same value as before - only age will differ).

---

### binaryInputStates

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: State (changing during operation of the device, but not persistently stored in the vDC) of a binary input.

**Array Element Properties**: See below for individual binary input state properties.

---

#### value (Binary State)

**Access**: Read  
**Type**: boolean or NULL  
**Required**: Yes  
**Parent**: binaryInputStates[N]  

**Description**: Current input state.

**Valid Values**:
- `false`: inactive
- `true`: active
- `NULL`: unknown state

---

#### extendedValue

**Access**: Read  
**Type**: integer or NULL  
**Required**: Yes  
**Parent**: binaryInputStates[N]  

**Description**: The property 'extendedValue' replaces the property 'value'. If the property extendedValue is present, the property value is not needed. The data from property 'value' is overwritten.

---

#### age (Binary State)

**Access**: Read  
**Type**: double or NULL  
**Required**: Yes  
**Parent**: binaryInputStates[N]  

**Description**: Age of the state shown in the value field in seconds. If no recent state is known, returns NULL.

---

#### error (Binary State)

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: binaryInputStates[N]  

**Description**: Error state of the binary input.

**Valid Values**:
- `0`: ok
- `1`: open circuit
- `2`: short circuit
- `4`: bus connection problem
- `5`: low battery in device
- `6`: other device error

---

## Sensor Input Properties

Virtual devices can have zero to several sensors. The following properties provide access to sensor input descriptions, settings, and states.

### sensorDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions of the sensors in the device.

**Array Element Properties**: See below for individual sensor description properties.

---

#### name (Sensor)

**Access**: Read  
**Type**: string  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: Human readable name/number for the sensor.

---

#### dsIndex (Sensor)

**Access**: Read  
**Type**: integer (0..N-1)  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: Index number where N=number of sensors in this device.

---

#### sensorType

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: Describes the type of physical unit the sensor measures.

**Valid Values**:
- `0`: none
- `1`: Temperature in °C
- `2`: Relative humidity in %
- `3`: Illumination in lux
- `4`: supply voltage level in V
- `5`: CO concentration in ppm
- `6`: Radon activity in Bq/m3
- `7`: gas type sensor
- `8`: particles <10µm in μg/m3
- `9`: particles <2.5µm in μg/m3
- `10`: particles <1µm in μg/m3
- `11`: room operating panel set point, 0..100%
- `12`: fan speed, 0..1 (0=off, <0=auto)
- `13`: Wind speed in m/s (average)
- `14`: Active Power in W
- `15`: Electric current in A
- `16`: Energy Meter in kWh
- `17`: Apparent Power in VA
- `18`: Air pressure in hPa
- `19`: Wind direction in degrees
- `20`: Sound pressure level in dB
- `21`: Precipitation intensity in mm/m2 (sum of last hour)
- `22`: CO2 concentration in ppm
- `23`: Wind gust speed in m/s
- `24`: Wind gust direction in degrees
- `25`: Generated Active Power in W
- `26`: Generated Energy in kWh
- `27`: Water Quantity in l
- `28`: Water Flow Rate in l/s

---

#### sensorUsage

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: Describes the usage field for the sensor.

**Valid Values**:
- `0`: undefined (generic usage or unknown)
- `1`: room
- `2`: outdoor
- `3`: user interaction (setting, dial)
- `4`: device level measurement (total, sum)
- `5`: device level last run
- `6`: device level average

---

#### min (Sensor)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: Minimum value.

---

#### max (Sensor)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: Maximum value.

---

#### resolution (Sensor)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: Resolution (size of LSB of actual HW sensor).

---

#### updateInterval (Sensor)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: How fast the physical value is tracked, in seconds, approximately. The purpose of this is to give information about the time resolution that can be expected from that sensor.

---

#### aliveSignInterval

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: sensorDescriptions[N]  

**Description**: How fast the sensor minimally sends an update. If sensor does not push a value for longer than that, it can be considered out-of-order.

---

### sensorSettings

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Settings (properties that can be changed and are stored persistently in the vDC) of the sensors in the device.

**Array Element Properties**: See below for individual sensor setting properties.

---

#### group (Sensor Settings)

**Access**: Read/Write  
**Type**: integer  
**Required**: Yes  
**Parent**: sensorSettings[N]  

**Description**: dS group number.

---

### sensorStates

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Value of a sensor. Changing during operation of the device, but not persistently stored in the vDC.

**Array Element Properties**: See below for individual sensor state properties.

---

#### value (Sensor State)

**Access**: Read  
**Type**: double or NULL  
**Required**: Yes  
**Parent**: sensorStates[N]  

**Description**: Current sensor value in the unit according to sensorType. If no recent state is known, returns NULL.

---

#### age (Sensor State)

**Access**: Read  
**Type**: double or NULL  
**Required**: Yes  
**Parent**: sensorStates[N]  

**Description**: Age of the state shown in the value field in seconds. If no recent state is known, returns NULL.

---

#### error (Sensor State)

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: sensorStates[N]  

**Description**: Error state of the sensor.

**Valid Values**:
- `0`: ok
- `1`: open circuit
- `2`: short circuit
- `4`: bus connection problem
- `5`: low battery in device
- `6`: other device error


---

## Output Properties

Devices with no output functionality return a NULL response when queried for outputDescription, outputSettings or outputState. Virtual devices in the digitalSTROM system can have a single output (or none at all, as for pure button devices). The output can have one or multiple channels.

**Important Notes**:
- Devices with no output at all do not have output- and channel-related properties.
- Output is meant as "output functionality" - like a lamp, a blind, a washing machine. Such outputs may need multiple physical parameters to control, and thus will likely have multiple physical/electrical output lines. These multiple parameters do not count as separate outputs, but are called channels (of the single output).
- If a physical device does have multiple, independent outputs (such as a multi-channel dimmer for example), a vDC must represent such a physical device as multiple virtual devices, with separate dSUIDs. The dSUID numbering scheme provides an enumeration field (17th byte) for that purpose.

### outputDescription

**Access**: Read  
**Type**: list of output description properties (container)  
**Required**: No (Optional)  

**Description**: Descriptions (invariable properties) of the device's output. Devices with no output don't have this property.

**Container Properties**: See below for individual output description properties.

---

#### function (Output)

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: outputDescription  

**Description**: Output function type.

**Valid Values**:
- `0`: on/off only (with channel 1, "brightness", switched on when "brightness">"onThreshold")
- `1`: dimmer (with channel 1, "brightness")
- `2`: positional (e.g. valves, blinds)
- `3`: dimmer with color temperature (with channels 1 and 4, "brightness", "ct")
- `4`: full color dimmer (with channels 1-6, "brightness", "hue", "saturation", "ct", "cieX", "cieY")
- `5`: bipolar, with negative and positive values (e.g. combined heating/cooling valve, in/out fan control)
- `6`: internally controlled (e.g. device has temperature control algorithm integrated)

---

#### outputUsage

**Access**: Read  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: outputDescription  

**Description**: Describes the usage field for the output (beyond device color).

**Valid Values**:
- `0`: undefined (generic usage or unknown)
- `1`: room
- `2`: outdoors
- `3`: user (display/indicator)

---

### outputSettings

**Access**: Read/Write  
**Type**: list of output settings properties (container)  
**Required**: No (Optional)  

**Description**: Settings (properties that can be changed and are stored persistently in the vDC) of the device's output. Devices with no output don't have this property.

**Container Properties**: See below for individual output setting properties.

---

#### mode (Output Settings)

**Access**: Read/Write  
**Type**: integer (enumeration)  
**Required**: Yes  
**Parent**: outputSettings  

**Description**: Output mode.

**Valid Values**:
- `0`: disabled, inactive
- `1`: binary
- `2`: gradual
- `127`: default (generically enabled using device's default mode)

---

### outputState

**Access**: Read/Write  
**Type**: list of output state properties (container)  
**Required**: No (Optional)  

**Description**: State (changing during operation of the device, but not persistently stored in the vDC) of the outputs. Devices with no output don't have this property.

---

## Channel Properties

Output channels represent the individual controllable parameters of a device's output. For example, a color light might have channels for brightness, hue, saturation, etc.

### channelDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions (invariable properties) of the channels in the device.

**Array Element Properties**: See below for individual channel description properties.

---

#### name (Channel)

**Access**: Read  
**Type**: string  
**Required**: Yes  
**Parent**: channelDescriptions[N]  

**Description**: Human readable name/number for the channel (e.g. matching labels for hardware connectors).

---

#### channelType

**Access**: Read  
**Type**: integer  
**Required**: Yes  
**Parent**: channelDescriptions[N]  

**Description**: Numerical Type Id of the channel. For definitions of output channel types please refer to the document ds-basics.pdf, section "Output Channels".

---

#### dsIndex (Channel)

**Access**: Read  
**Type**: integer (0..N-1)  
**Required**: Yes  
**Parent**: channelDescriptions[N]  

**Description**: Index number where N=number of channels in this device. The index is used for dS-OS and DSMAPI addressing. Index "0" is by definition the default output channel.

---

#### min (Channel)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: channelDescriptions[N]  

**Description**: Minimum value.

---

#### max (Channel)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: channelDescriptions[N]  

**Description**: Maximum value.

---

#### resolution (Channel)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: channelDescriptions[N]  

**Description**: Resolution.

---

### channelSettings

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Settings (properties that can be changed and are stored persistently in the vDC) for the device's channels.

**Note**: Currently there are no per-channel settings defined.

---

### channelStates

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Current state of the device's channels.

**Important**: Channel State must not be written to. Instead, the notification setOutputChannelValue must be used.

**Array Element Properties**: See below for individual channel state properties.

---

#### value (Channel State)

**Access**: Read  
**Type**: double  
**Required**: Yes  
**Parent**: channelStates[N]  

**Description**: Current channel value (brightness, blind position, power on/off).

**Important**: Read-only. Use setOutputChannelValue to change channel values.

---

#### age (Channel State)

**Access**: Read  
**Type**: double or NULL  
**Required**: Yes  
**Parent**: channelStates[N]  

**Description**: Age of the state shown in the value field in seconds. This indicates when the value was last applied to the actual device hardware, or when an actual output status was last received from the device. Age is NULL when a new value was set, but not yet applied to the device.

---

## Scene Properties

A scene stores a set of values to apply to the outputs of the device when a particular scene is called. This property is available on devices having at least one output with standard behaviour defined, such as light, or shadow. Input-only devices do not support this property.

### scenes

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: The scene table of the device. Represented as a list of property elements, one for each scene. The property elements are named by scene number, starting with "0".

**Array Element Properties**: See below for individual scene properties.

---

#### channels (Scene)

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: Yes  
**Parent**: scenes[N]  

**Description**: For each channel, a scene value (consisting of value and dontCare flag). Represented as a list of property elements, one for each channel in the device. The property elements are named by channel type id (which can be 0 for devices controlling an unspecified functionality, such as a generic switch output).

**Array Element Properties**: See below for individual scene channel properties.

---

#### value (Scene Channel)

**Access**: Read/Write  
**Type**: double  
**Required**: Yes  
**Parent**: scenes[N]/channels[M]  

**Description**: The value for the related channel. The value range and resolution is the same as for the related channel's channelState value property.

---

#### dontCare

**Access**: Read/Write  
**Type**: boolean  
**Required**: Yes  
**Parent**: scenes[N]/channels[M]  

**Description**: Channel-specific dontCare flag: if set, calling this scene does not apply the stored channel value (but possibly other channel's values which don't have dontCare set).

**Note**: There is also a scene-global dontCare flag that can be set at the scene level.

---

#### automatic

**Access**: Read/Write  
**Type**: boolean  
**Required**: Yes  
**Parent**: scenes[N]/channels[M]  

**Description**: Channel-specific automatic flag: if set, calling this scene activates the internal automatic control logic.

---

## Action Properties

Action descriptions describe basic functionalities and operation processes of a device. They serve as a template to create custom defined actions as variation of templates with modified parameter sets.

### deviceActionDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions of the available action methods in the device. These are the basic device actions methods (invariable).

**Array Element Properties**: Each action description contains:
- `name` (mandatory, string): name of this action property entry
- `params` (optional, list of Parameter Objects): parameter list related to this action
- `description` (optional, string): description of this template action

---

### customActions

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions of the user defined actions methods in the device. Custom actions are configured by the user. They can be created via API and are persistently stored on the VDC.

**Array Element Properties**: Each custom action contains:
- `name` (mandatory, string): unique id of this custom action property entry, always has prefix `custom.`
- `action` (mandatory, string): reference id of the template action, on which this standard action is based upon
- `title` (mandatory, string): human readable name of this custom action, in most cases given by the user
- `params` (optional, list of Parameter Name:Value pairs): list of parameter values that are different to the template action

---

## State and Property Descriptions

Device State represent a status within a device. States differ from Properties in the way that they have limited number of possible values, whereas Properties are more generic and do have limitations on their value, with respect to their type.

### deviceStateDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions of the available state objects in the device. State changes are signaled along with the new state value.

**Array Element Properties**: Each state description contains:
- `name` (mandatory, string): name of this state property entry
- `options` (mandatory, list of Option Id:Value pairs): option list related to this state, e.g. 0: Off, 1: Initializing, 2: Running, 3: Shutdown
- `description` (optional, string): description of this state

---

### deviceStates

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Value of the state objects.

**Array Element Properties**: Each state value contains:
- `name` (mandatory, string): name of this state property entry
- `value` (mandatory, string): option value

---

### devicePropertyDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions of the available property objects in the device.

**Array Element Properties**: Each property description contains:
- `name` (mandatory, string): name of this property entry
- `type` (mandatory, numeric/enumeration/string): data type of the property value
- `min` (optional, numeric only, double): minimum value
- `max` (optional, numeric only, double): maximum value
- `resolution` (optional, numeric only, double): resolution (size of LSB of actual HW sensor)
- `siunit` (optional, numeric only, string): The SI Unit as a string, incl. prefixes like kilo or milli
- `options` (optional, enumeration only, list of key:value pairs): the option values for the enumeration
- `default` (optional, all types, double/string/option): the default value of this property

---

### deviceProperties

**Access**: Read/Write  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Value of the property objects.

**Array Element Properties**: Each property value contains:
- `name` (mandatory, string): name of this state property entry
- `value` (mandatory, string): property value

---

## Event Properties

Device Events are state-less events that can occur within a device.

### deviceEventDescriptions

**Access**: Read  
**Type**: list of property elements (array)  
**Required**: No (Optional)  

**Description**: Descriptions of the available events in the device.

**Array Element Properties**: Each event description contains:
- `name` (mandatory, string): name of this event property entry
- `description` (optional, string): description of this event

---

## Summary

This comprehensive property element reference provides detailed information about every property available in the vDC API property tree system. Use this document as your authoritative reference when implementing vDC hosts or clients.

**Key Takeaways**:
1. Properties are organized hierarchically into trees
2. vDC and device (vdSD) entities have different property sets
3. Many properties are common between entity types
4. Array properties use string indices ("0", "1", etc.)
5. Always check whether a property is required or optional
6. Respect read-only vs. read-write access restrictions

**Related Documentation**:
- [Property System Guide](./04-property-system.md) - Understanding property trees and how to work with them
- [vDC Reference](./05-vdc-reference.md) - vDC level properties
- [vdSD Reference](./06-vdsd-reference.md) - Device properties
- [Device Features](./07-device-features.md) - Inputs, outputs, scenes

---

[← Back: Property System](./04-property-system.md) | [Back to Index](./README.md) | [Next: vDC Reference →](./05-vdc-reference.md)
