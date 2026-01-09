# 4 Virtual digitalSTROM device (vdSD) properties

- The following table applies to entities which have a value of "vdSD" for the "type" property.
- All vdSDs must also support the basic set of properties as described under 2 "Common properties" above.

## 4.1 Properties on the vdSD level

### 4.1.1 General device properties

| Property Name | acc | Type / Range | Description |
|---|---|---|---|
| primaryGroup | r | integer, dS class number | basic class (color) of the device |
| zoneID | r/w | integer, global dS Zone ID | this should be updated by the vdSM to reflect the zone the device is in. The vDC may use this value to optimize zone calls (i.e. bundle calls to actual hardware if single device calls are slow) |
| progMode | r/w | optional boolean | enables local programming mode (for those devices that have it) |
| modelFeatures | r | list of property elements | Descriptions (invariable properties) of the device model features.<br>• each available feature is represented as a property element (key/value) with a boolean true value. Not available features are not included in the list.<br>• this property represents the virtual device's row in the "visibility Matrix", and determines what dSS configurator UI features (dialogs, settings, controls) the device will have. |
| currentConfigId | r | optional string | Configuration or profile ID that is currently activate in the dS-Device. The current active configuration can be changed with the "setConfiguration" generic request message. |
| configurations | r | list of property elements | List of configuration or profiles ID's supported by this device. |
