## 4.9 Output Channel

### 4.9.1 Output Channel Description

Description (invariable properties) of the device's channels. The properties described here are contained in the device-level 4.1.3 channelDescriptions property.

| Property Name | acc | Type / Range | Description |
|---|---|---|---|
| name | r | string | human readable name/number for the channel (e.g. matching labels for hardware connectors) |
| channelType | r | integer | Numerical Type Id of the channel. For definitions of output channel types please refer to the document ds-basics.pdf, section "Output Channels". |
| dsIndex | r | integer | 0..N-1, where N=number of channels in this device. The index is used for dS-OS and DSMAPI addressing. Index "0" is by definition the default output channel. |
| min | r | double | min value |
| max | r | double | max value |
| resolution | r | double | resolution |

The following attributes shall no longer be used (Version 1.0.2):

| Property Name | acc | Type / Range | Description |
|---|---|---|---|
| channelIndex | r | integer | 0..N-1, where N=number of channels in this device. The index is device specific and no assumption on any particular order of indexes vs. channel types must be made. |

### 4.9.2 Output Channel Settings

Settings (properties that can be changed and are stored persistently in the vDC) for the device's channels. The properties described here are contained in the device-level 4.1.3 channelSettings property.

**Notice** Currently there are no per-channel settings defined

| Property Name | acc | Type / Range | Description |
|---|---|---|---|
| - | - | - | - |

### 4.9.3 Output Channel State

Current state of the device's channels. The properties described here are contained in the device-level 4.1.3 channelStates property.

**Notice** Channel State must not be written to. Instead, the notification setOutputChannelValue must be used.

| Property Name | acc | Type / Range | Description |
|---|---|---|---|
| value | r | double | current channel value (brightness, blind position, power on/off) |
| age | r | double | age of the state shown in the value field in seconds. This indicates when the value was last applied to the actual device hardware, or when an actual output status was last received from the device. age is NULL when a new value was set, but not yet applied to the device |
