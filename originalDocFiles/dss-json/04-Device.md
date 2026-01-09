4 Device
========

4.1
---

Common

Every /json/device/ function uses a common selection scheme for the device to which the command

refers to. Either the parameter ”dsid” or ”name” must be given to identify the device.

| Parameter | Description | Remarks |
|---|---|---|
| dsid | DevicedSIDString | Optional |
| name | DeviceName | Optional |
| category | RequestCategory | Optional |

Device dSID String Optional

A missing device identifier result in the following error message to be returned.

```json
{  ”ok” :  false ,  ”message” :  ”Need￿parameter￿name￿or￿dsid￿to￿identify￿device”  }
```

If a device identifier does not match any actually known device in the installation the following error

message is returned.

```json
{  ”ok” :  false ,  ”message” :  ”Could￿not￿find￿device￿named￿’Wandlampe￿am￿Eingang’”  }
```

The category parameter has an influence on how particular requests are treated, the goal is to prevent

scene calls from automated scripts in certain situations. Currently supported categories are:

• manual

• timer

• algoirthm

A missing category parameter is currently treated as manual category, this compatibility will be re-

moved in release 1.8.

4.2
---

4.2.1
-----

getName

Returns the user defined name of a device.

**Synopsis**

```json
HTTP GET /json/device/getName
```

None

**Response**

```json
HTTP Status 200
```

result.name identifier string for the device

**Sample**

```json
GET /json/device/getName?dsid=3504175fe000000000017ef3
{
”ok” : true ,
”result”  :
{
”name”  :  ”App − Taster”
```

```json
}
}
```

4.2.2
-----

setName

Sets the device name.

**Synopsis**

```json
HTTP GET /json/device/setName
```

| Parameter | Description | Remarks |
|---|---|---|
| newName | identifierstringforthedevice | Mandatory |

identifier string for the device Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setName?id=3504175fe000000000017ef3&newName= ”Wohnen”
{
”ok” : true
}
```

4.2.3
-----

getSpec

Retrieves device and product information.

**Synopsis**

```json
HTTP GET /json/device/getSpec
```

None

**Response**

```json
HTTP Status 200
```

result.functionID Function ID of the device

result.productID Product ID of the device

result.revisionID Revision ID of the device

**Sample**

```json
GET /json/device/getName?dsid=3504175fe000000000017ef3
{
”ok” :  true ,
”result” : {
”functionID” : 33027,
”productID” : 1224,
```

```json
”revisionID” : 834
}
}
```

4.3
---

First seen

4.3.1
-----

getFirstSeen

Returns the timestamp when the device was registered.

**Synopsis**

```json
HTTP GET /json/device/getFirstSeen
```

Parameter

None

**Response**

```json
HTTP Status 200
```

result.time ISO8601 time when device was registered

**Sample**

```json
GET /json/device/getFirstSeen?dsid=3504175fe000000000017ef3
{
”result”  :
{
”time”  :  ”2010 − 10 − 01T10:42:13Z”
},
”ok” : true
}
```

4.4
---

Groups

4.4.1
-----

getGroups

Returns a list of groups the device is registered in.

**Synopsis**

```json
HTTP GET /json/device/getGroups
```

Parameter

None

**Response**

```json
HTTP Status 200
```

result.groups array of groups of the device

**Sample**

```json
GET /json/device/getGroups
{
”ok” :  true ,
”result” : {
”groups” : [
{
”id” : 3,
”name” :  ”blue”
},
{
”id” : 8,
”name” :  ”black”
}
]
}
}
```

4.5
---

Scene

4.5.1
-----

callScene

Excutes the scene  sceneNumber  on a devices.

**Synopsis**

```json
HTTP GET /json/device/callScene
```

sceneNumber Numerical value

Boolean value, if set a forced scene call is issued Optional

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/callScene?dsid=3504175fe000000000017ef3&sceneNumber=13
{
”ok” : true
}
```

4.5.2
-----

saveScene

Tells the device to store the current output values as a default for the scene  sceneNumber .

**Synopsis**

```json
HTTP GET /json/device/saveScene
```

sceneNumber Numerical value Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/saveScene?dsid=3504175fe000000000017ef3&sceneNumber=5
{
”ok” : true
}
```

4.5.3
-----

undoScene

Tells devices to restore the output values to the previous state if the current scene matches the  sce-

neNumber .

**Synopsis**

```json
HTTP GET /json/device/undoScene
```

sceneNumber Numerical value Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/undoScene?dsid=3504175fe000000000017ef3&sceneNumber=18
{
”ok” : true
}
```

4.5.4
-----

turnOn

Tells devices to execute the scene MAX.

**Synopsis**

```json
HTTP GET /json/device/turnOn
```

None

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/turnOn?dsid=3504175fe000000000017ef3
{
”ok” : true
}
```

4.5.5
-----

turnOff

Tells devices to execute the scene MIN.

**Synopsis**

```json
HTTP GET /json/device/turnOff
```

Parameter

None

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/turnOff?dsid=3504175fe000000000017ef3
{
”ok” : true
}
```

4.5.6
-----

increaseValue

Tells devices to execute the scene INC.

**Synopsis**

```json
HTTP GET /json/device/increaseValue
```

Parameter

None

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/increaseValue?dsid=3504175fe000000000017ef3
{
”ok” : true
}
```

4.5.7
-----

decreaseValue

Tells devices to execute the scene DEC.

**Synopsis**

```json
HTTP GET /json/device/decreaseValue
```

None

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/decreaseValue?dsid=3504175fe000000000017ef3
{
”ok” : true
}
```

4.6
---

Value

4.6.1
-----

Set Value

Set the primary output value of a device to a given value.

Notice  Setting output values directly bypasses the group state machine and is unrecommended.

**Synopsis**

```json
HTTP GET /json/device/setValue
```

| Parameter | Description | Remarks |
|---|---|---|
| value | Numerical8bitvalue,intherangefrom0to255 | Mandatory |

Numerical 8 bit value, in the range from 0 to 255 Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setValue?dsid=3504175fe000000000017ef3&value=127
{
”ok” : true
}
```

4.6.2
-----

Set Output Value

Set a output channel value of a device to a given value. The available output parameter ranges and

channels depend on the feature of the hardware components.

Notice  Setting output values directly bypasses the group state machine and is unrecommended.

**Synopsis**

```json
HTTP GET /json/device/setOutputValue
```

| Parameter | Description | Remarks |
|---|---|---|
| value | NumericalValue | Mandatory |
| offset | OutputDataOffset | Mandatory |

Output Data Offset Mandatory

offset Description

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setOutputValue?dsid=3504175fe000000000017ef3&offset=2&value=5345
{
”ok” : true
}
```

4.6.3
-----

Get Output Value

Get the current output channel status of a device. The available output channels depend on the feature

of the hardware components.

Notice  Getting output values directly from the device takes a noticeable amount of time. This re-

quest is subject of limitations in the systems certification rules.

**Synopsis**

```json
HTTP GET /json/device/getOutputValue
```

| offset | Description | Product |
|---|---|---|
| 2 | shadeOpeningAngleOutside,16Bit | GR-KL220,GR-HKL230 |

Output Data Offset Mandatory

Named Parameter Mandatory

Either  offset  or  type  parameter has to be given.

GR-KL200, GR-KL210,

GR-KL200, GR-KL210,

currentOutputMode output operation mode when automatic selec-

**Response**

```json
HTTP Status 200
```

result.offset the given offset from the request

result.value Numerical value of the selected output channel queried from the device

**Sample**

```json
GET /json/device/getOutputValue?dsid=3504175fe000000000017ef3&offset=0
{
”ok” :  true ,  ”result” : {  ”offset” : 0,  ”value” : 5345 }
}
```

4.6.4
-----

Get Scene Value

Retrieves the device value of the given scene.

**Synopsis**

```json
HTTP GET /json/device/getSceneValue
```

| type | Description | Product |
|---|---|---|
| position | Targetposition,16Bit | GR-KL200, GR-KL210, |
| angle | Targetangleposition,8Bit | GR-KL220,GR-KL230 |
| positionCurrent | Currentpositionwhenactive,16Bit | GR-KL220,GR-KL230 |
| pwmValue | CurrentPWMvalue,8Bit | GR-KL200, GR-KL210, |
| pwmPriorityMode | CurrentPWMpriorityvalue | GR-KL220,GR-KL230 |
| currentOutputMode | outputoperationmodewhenautomaticselec- | BL-KM200, BL- |
|  | tionisactive | SDS200 |
|  |  | BL-KM200, BL- |
|  |  | SDS200 |
|  |  | GE-KM300, GE- |
|  |  | TKM300 |

Numerical value Mandatory

**Response**

```json
HTTP Status 200
```

result.scenes if available, a json object with a command field that is either a standard or custom device action

**Sample**

```json
GET /json/device/getSceneValue?dsuid=3504175fe0000000000000017ef300&sceneID=72
{
”ok” : true , ”result” :{ ”value” :65535, ”angle” :255}
}
GET /json/device/getSceneValue?dsuid=687ba4e345e75bd58093bf119f8a6c6700&sceneID=72
{
”ok” : true , ”result” :{ ”value” :0}
}
GET /json/device/getSceneValue?dsuid=df6aa5bba4db5540c0fe55e3eb088be900&sceneID=72
”result” : {
”scenes” : {
”72” : {
”channels” :  null ,
”command” :  ”std.stop” ,
”dontCare” :  false ,
”effect” : 1,
”ignoreLocalPriority” :  true
}
}
},
”ok” :  true
}
```

4.6.5
-----

Set Scene Value

Retrieves the device value of the given scene.

**Synopsis**

```json
HTTP GET /json/device/setSceneValue
```

| Parameter | Description | Remarks |
|---|---|---|
| sceneID | Numericalvalue | Mandatory |
| value | Numericalvalue | Mandatory |
| angle | Numericalvalue,ifapplicable | Optional |
| command | Stringvalue,ifapplicable | Optional |

Numerical value, if applicable Optional

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setSceneValue?dsuid=3504175fe0000000000000016c4f00&sceneID=72&value=26987
{
”ok” : true
}
GET /json/device/setSceneValue?dsuid=687ba4e345e75bd58093bf119f8a6c6700&sceneID=81&value=100&command=boilandcooldown
{
”ok” : true
}
```

4.6.6
-----

Get Scene Mode

Reads the device configuration flags for a given  sceneID . For details about the scene configuration see

the ds-basics reference document.

**Synopsis**

```json
HTTP GET /json/device/getSceneMode
```

| Parameter | Description | Remarks |
|---|---|---|
| sceneID | Scenenumberforwhichtheconfigurationisrequested | Mandatory |

Scene number for which the configuration is requested Mandatory

**Response**

```json
HTTP Status 200
```

dimmTimeIndex Index of the transition configuration register

**Sample**

```json
GET /json/device/getSceneMode?dsid=3504175fe000000000016be7&sceneID=5
{
”ok” :  true ,
”result” :
{
”sceneID” : 5,
”dontCare” :  false ,
”localPrio” :  false ,
”specialMode” :  false ,
”flashMode” :  false ,
”ledconIndex” : 1,
”dimtimeIndex” : 1
}
}
```

4.6.7
-----

Set Scene Mode

Sets the device configuration flags for a given  sceneID . For details about the scene configuration see

the ds-basics reference document.

**Synopsis**

```json
HTTP GET /json/device/setSceneMode
```

dimmtimeIndex Index of the transition configuration register Optional

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setSceneMode?dsid=3504175fe000000000016be7&sceneID=5&dimtimeIndex=2&dontCare= true
{
”ok” :  true
}
```

4.6.8
-----

Blink

Executes the ”blink” function on a device for identification purposes.

**Synopsis**

```json
HTTP GET /json/device/blink
```

None

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/blink?dsid=3504175fe000000000017ef3
{
”ok” : true
}
```

4.6.9
-----

Get Output Channel Value

Retrieve the value of one or more output channels of the device.

**Synopsis**

```json
HTTP GET /json/device/getOutputChannelValue
```

| Parameter | Description | Remarks |
|---|---|---|
| channels | Semicolonseparatedlistofchannelnames | Mandatory |

Semicolon separated list of channel names Mandatory

Currently supported channels are listed below. For details please refer to the the dS-Basics document

in the section  Output Channel Types .

• brightness: light brightness

• hue: colored light hue

• saturation: colored light saturation

• colortemp: color temperature

• x: CIE color model x component

• y: CIE color model y component

• shadePositionOutside: shade position opening percentage for e.g. blinds and roller shutters

• shadePositionIndoor: shade position opening percentage for e.g. curtains

• shadeOpeningAngleOutside: shade position opening angle for e.g. lamellars

• shadeOpeningAngleIndoor: indoor shade position opening angle for e.g. lamellars

• transparency: transparency of e.g. a smart window

• airFlowIntensity: intensity of ventilation

• airFlowDirection: direction of air flow

• airFlapPosition: flap position

• airLouverPosition: louver position

• heatingPower: heating power and intensity

• coolingCapacity: cooling capacity and intensity

• audioVolume: audio loudness

• powerState: power status

**Response**

```json
HTTP Status 200
```

result.channels[x].channel output channel name

**Sample**

```json
GET /json/device/getOutputChannelValue?dsid=3504175fe000000000016c4f&channels=brightness;saturation
{
”ok” : true ,
result: {
channels: [
{ channel:  ”brightness” , value: 50 },
{ channel:  ”saturation” , value: 80 }
]
}
}
```

4.6.10
------

Set Output Channel Value

Set the value of one or more output channels of the device.

**Synopsis**

```json
HTTP GET /json/device/setOutputChannelValue
```

channelvalues Semicolon separated list of channel names

Immediately apply the new values to the

1 (true) by

See getOutputChannelValue description  4.6.9  for a list of output channel names and their value ranges.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setOutputChannelValue?dsid=3504175fe000000000016c4f&channelvalues=brightness=10;saturation=100&applyNow=1
{  ”ok” : true  }
```

4.6.11
------

Get Output Channel Value v2

Retrieve the current output value of selected or all channels of the device.

**Synopsis**

```json
HTTP GET /json/device/getOutputChannelValue2
```

| Parameter | Description | Remarks |
|---|---|---|
| channels | Semicolonseparatedlistofchannelnames | Optional |

Semicolon separated list of channel names Optional

If channels parameter is empty or omitted the call returns current values for all channels.

See getOutputChannelValue description  4.6.9  for a list of output channel names and their value ranges.

**Response**

```json
HTTP Status 200
```

result.channels[x].channel output channel name

**Sample**

```json
GET /json/device/getOutputChannelValue2?dsuid=5a11caa06212578280d826428d15c3d700&channels=brightness;saturation;hue
{
”ok” : true ,
result: {
channels: {
”brightness” : {  ”value” : 50,  ”automatic” :  false  },
”saturation” : {  ”value” : 80 },
”hue” : {  ”value” : 0 }
}
}
}
```

4.6.12
------

Set Output Channel Value v2

Set the value of one or all output channels of the device.

**Synopsis**

```json
HTTP GET /json/device/setOutputChannelValue2
```

| result.channels | arrayofchannelsandtheirvalues |
|---|---|
| result.channels[x].channel | outputchannelname |
| result.channels[x].value | outputchannelvalue |

1 (true) by

The  channels  parameter json object has the same structure like returned by getOutputChannelValue2.

See getOutputChannelValue description  4.6.9  for a list of output channel names and their value ranges.

WARNING: Values passed to  setOutputChannelValue2  are not symmetric to values in  getOutputChannel-

Value2  response. Client is responsible to scale the values passed to  setOutputChannelValue2  to units

used at dsm-api as specified in  ds-basics.pdf  section  Output Channel Types .

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setOutputChannelValue2?dsuid=5a11caa06212578280d826428d15c3d700&channels={ ”brightness” : { ”value” : 10,
”automatic” :  false },  ”saturation” : { ”value” : 100},  ”hue” : { ”value” : 235}}
{  ”ok” : true  }
```

4.6.13
------

Get Output Channel Scene Value

Get scene value of one or more output channels.

**Synopsis**

```json
HTTP GET /json/device/getOutputChannelSceneValue
```

sceneNumber Number of the scene for which the values

See getOutputChannelValue description  4.6.9  for a list of output channel names and their value ranges.

**Response**

```json
HTTP Status 200
```

result.channels[x].channel output channel name

**Sample**

```json
GET /json/device/getOutputChannelSceneValue?dsid=3504175fe000000000016c4f&channels=brightness;saturation&sceneNumber=1
{
”ok” : true ,
result: {
sceneID: 1,
```

```json
channels: [
{ channel:  ”brightness” , value: 40 },
{ channel:  ”saturation” , value: 20 }
]
}
}
```

4.6.14
------

Set Output Channel Scene Value

Set scene value of one or more output channels.

**Synopsis**

```json
HTTP GET /json/device/setOutputChannelSceneValue
```

channelvalues Semicolon separated list of channel names

sceneNumber Number of the scene for which the values

See getOutputChannelValue description  4.6.9  for a list of output channel names and their value ranges.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setOutputChannelSceneValue?dsid=3504175fe000000000016c4f&channelvalues=brightness=10;saturation=100&
sceneNumber=1
{
”ok” : true
}
```

4.6.15
------

Get Output Channel Scene Value v2

Reads the device configuration for a given  sceneID  and output channels.

**Synopsis**

```json
HTTP GET /json/device/getOutputChannelSceneValue2
```

sceneNumber Scene number for which the configuration is requested Mandatory

See getOutputChannelValue description  4.6.9  for a list of output channel names and their value ranges.

**Response**

```json
HTTP Status 200
```

channels.[x].automatic Automatic Operation Flag

**Sample**

```json
GET /json/device/getOutputChannelSceneValue2?dsid=3504175fe000000000016be7&sceneNumber=5&
channels=airFlowIntensity;airLouverPosition
{
”ok” :  true ,
”result” :
{
”sceneID”  : 5,
”channels”  : {
”airFlowIntensity”  : {  ”value” : 0,  ”dontCare” :  false ,  ”automatic” :  true },
”airLouverPosition”  : {  ”value” : 50,  ”dontCare” :  false ,  ”automatic” :  false }
}
}
}
```

4.6.16
------

Set Output Channel Scene Value v2

Sets the device configuration flags for a given  sceneID  and output channels.

**Synopsis**

```json
HTTP GET /json/device/setOutputChannelSceneValue2
```

sceneNumber Scene number which has been requested

The  channels  parameter json object has the same structure like returned by getOutputChannelSceneValue2.

See getOutputChannelValue description  4.6.9  for a list of output channel names and their value ranges.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setOutputChannelSceneValue2?dsid=3504175fe000000000016be7&sceneNumber=5&channels={  ”airFlowIntensity”  :
{ ”dontCare” :  true },  ”airLouverPosition”  : { ”value” : 100,  ”automatic” :  true }}
{
”ok” :  true
}
```

4.6.17
------

Get Output Channel Don’t Care Flags

Get don’t care flags for one or more output channels.

Notice  getOutputChannelDontCareFlag is DEPRECATED. Please use  Get Output Channel Scene

Mode  instead.

**Synopsis**

```json
HTTP GET /json/device/getOutputChannelDontCareFlags
```

sceneNumber Scene number for which the flag will be set Mandatory

**Response**

```json
HTTP Status 200
```

result.channels[x].dontCare output channel ”don’t care” flag value

**Sample**

```json
GET /json/device/getOutputChannelDontCareFlags?dsid=3504175fe000000000016c4f&channels=brightness;saturation&dontCare=1&
sceneNumber=1
{
”ok” : true ,
result: {
channels: [
{ channel:  ”brightness” , dontCare: 0 },
{ channel:  ”saturation” , dontCare: 1 }
}
}
```

4.6.18
------

Set Output Channel Don’t Care Flag

Set don’t care flag for one or more output channels.

Notice  setOutputChannelDontCareFlag is DEPRECATED. Please use  Set Output Channel Scene

Mode  instead.

**Synopsis**

```json
HTTP GET /json/device/setOutputChannelDontCareFlag
```

Semicolon separated list of channel names Mandatory

sceneNumber Scene number for which the flag will be set Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setOutputChannelDontCareFlag?dsid=3504175fe000000000016c4f&channels=brightness;saturation&dontCare=1&
sceneNumber=1
{
”ok” : true
}
```

4.7
---

Configuration

4.7.1
-----

setButtonID

Sets the button ID of a device. For details about the push button configuration see the ds-basics refer-

ence document.

**Synopsis**

```json
HTTP GET /json/device/setButtonID
```

| Parameter | Description | Remarks |
|---|---|---|
| channels | Semicolonseparatedlistofchannelnames | Mandatory |
| dontCare | Don’tcareflagvalue,boolean | Mandatory(0or1) |
| sceneNumber | Scenenumberforwhichtheflagwillbeset | Mandatory |

Button number to set Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setButtonID?dsid=3504175fe000000000016be7&buttonID=5
{
”ok” :  true
}
```

4.7.2
-----

setButtonInputMode

Sets the button input mode of a device. For details about the push button configuration see the ds-

basics reference document.

**Synopsis**

```json
HTTP GET /json/device/setButtonInputMode
```

| Parameter | Description | Remarks |
|---|---|---|
| modeID | Numericalvalueofthebuttonmodetoset | Mandatory |

Numerical value of the button mode to set Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setButtonInputMode?dsid=3504175fe000000000016be7&modeID=0
{
”ok” :  true
}
```

4.7.3
-----

setOutputMode

Sets the output mode of a device.

**Synopsis**

```json
HTTP GET /json/device/setOutputMode
```

| ok | true |
|---|---|

Numerical value of the output mode to set Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setOutputMode?dsid=3504175fe000000000016be7&modeID=0
{
”ok” :  true
}
```

4.7.4
-----

setJokerGroup

Sets the color group of a Joker device.

**Synopsis**

```json
HTTP GET /json/device/setJokerGroup
```

| Parameter | Description | Remarks |
|---|---|---|
| groupID | Groupnumbertoset | Mandatory |

Group number to set Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setJokerGroup?dsid=3504175fe000000000016be7&groupID=2
{
”ok” :  true
}
```

4.7.5
-----

setButtonActiveGroup

Sets the user group of a push button device.

**Synopsis**

```json
HTTP GET /json/device/setButtonActiveGroup
```

| ok | true |
|---|---|

Group number to set Mandatory, value range between 0 and 63,use 0xff to reset

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setButtonActiveGroup?dsid=3504175fe000000000016be7&groupID=20
{
”ok” :  true
}
```

4.7.6
-----

getTransitionTime

Reads the device transition time configuration for a given register set. For details about the transition

time configuration see the ds-basics reference document.

**Synopsis**

```json
HTTP GET /json/device/getTransitionTime
```

dimtimeIndex Index of the transition configuration register Mandatory

**Response**

```json
HTTP Status 200
```

dimmtimeIndex Index of the transition configuration register

**Sample**

```json
GET /json/device/getTransitionTime?dsid=3504175fe000000000016be7&dimtimeIndex=2
{
”ok” :  true ,
”result” :
{
”dimtimeIndex” : 2,
”up” : 600,
”down” : 55
}
}
```

4.7.7
-----

setTransitionTime

Sets the device transition time configuration for a given register set. For details about the transition

time configuration see the ds-basics reference document.

**Synopsis**

```json
HTTP GET /json/device/setTransitionTime
```

dimtimeIndex Index of the transition configuration register Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setTransitionTime?dsid=3504175fe000000000016be7&dimtimeIndex=2&up=600&down=600
{
”ok” :  true
}
```

4.7.8
-----

setConfig

Write a configuration value of a config class parameter to the device.

Notice  Writing configuration parameters directly to the device may lead to malfunctions including

complete failure of the whole device. Do not write parameters or values unless you are sure that

the device supports it.

**Synopsis**

```json
HTTP GET /json/device/setConfig
```

| Parameter | Description | Remarks |
|---|---|---|
| dimtimeIndex | Indexofthetransitionconfigurationregister | Mandatory |
| up | RamptimeupinMilliseconds | Mandatory |
| down | RamptimedowninMilliseconds | Mandatory |

Configuration Class Mandatory

**Response**

```json
HTTP Status 200
```

class the class parameter from the request

index the index parameter from the request

value parameter value

**Sample**

```json
GET /json/device/setConfig?dsid=3504175fe000000000016be7& class =3&index=0&value=33
{
”ok” :  true
}
```

4.7.9
-----

getConfig

Reads a 8 bit parameter value of a config class from the device.

Notice  Getting parameter values directly from the device takes a noticeable amount of time. This

request is subject of limitations in the systems certification rules.

**Synopsis**

```json
HTTP GET /json/device/getConfig
```

| Parameter | Description | Remarks |
|---|---|---|
| class | Configurationclass | Mandatory |
| index | Parameterindex | Mandatory |

Configuration class Mandatory

**Response**

```json
HTTP Status 200
```

class the class parameter from the request

index the index parameter from the request

value parameter value

**Sample**

```json
GET /json/device/getConfig?dsid=3504175fe000000000016be7& class =1&index=2
{
”ok” :  true ,
”result” :
{
”class” : 1,
”index” : 2,
”value” : 231
}
}
```

4.7.10
------

getConfigWord

Reads a 16 bit parameter value of a config class from the device.

Notice  Getting parameter values directly from the device takes a noticeable amount of time. This

request is subject of limitations in the systems certification rules.

**Synopsis**

```json
HTTP GET /json/device/getConfigWord
```

| Parameter | Description | Remarks |
|---|---|---|
| class | Configurationclass | Mandatory |
| index | Parameterindex,even | Mandatory |

Parameter index, even Mandatory

**Response**

```json
HTTP Status 200
```

class the class parameter from the request

index the index parameter from the request

value parameter value

**Sample**

```json
GET /json/device/getConfigWord?dsid=3504175fe000000000016be7& class =3&index=2
{
”ok” :  true ,
”result” :
{
”class” : 3,
”index” : 2,
”value” : 65280
}
}
```

4.7.11
------

setCardinalDirection

Write the cardinal direction of the device.

**Synopsis**

```json
HTTP GET /json/device/setCardinalDirection
```

| class index value | theclassparameterfromtherequest theindexparameterfromtherequest parametervalue |
|---|---|

the cardinal direction of this device. Allowed values:

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/device/setCardinalDirection?dsid=3504175fe000000000016be7&direction=south\%20west
{
”ok” :  true
}
```

4.7.12
------

getCardinalDirection

Read the configured cardinal direction of the device.

**Synopsis**

```json
HTTP GET /json/device/getCardinalDirection
```

**Response**

```json
HTTP Status 200
```

direction the cardinal direction of this device. Allowed values:

**Sample**

```json
GET /json/device/getCardinalDirection?dsid=3504175fe000000000016be7
{
”ok” :  true ,
”result” :
{
”direction” :  ”south￿west”
}
}
```

4.7.13
------

setWindProtectionClass

Write the wid protection class of the device.

**Synopsis**

```json
HTTP GET /json/device/setWindProtectionClass
```

| Parameter | Description | Remarks |
|---|---|---|
| class | thewindprotectionclassofthisdevice. |  |

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/device/setWindProtectionClass?dsid=3504175fe000000000016be7& class =4
{
”ok” :  true
}
```

4.7.14
------

getWindProtectionClass

Read the the wid protection class of the device.

**Synopsis**

```json
HTTP GET /json/device/getWindProtectionClass
```

**Response**

```json
HTTP Status 200
```

class the wind protection class of this device.

**Sample**

```json
GET /json/device/getWindProtectionClass?dsid=3504175fe000000000016be7
{
”ok” :  true ,
”result” :
{
”class” : 2
}
}
```

4.7.15
------

setFloor

Write floor number where the device is installed.

**Synopsis**

```json
HTTP GET /json/device/setFloor
```

| class | thewindprotectionclassofthisdevice. |
|---|---|

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/device/setFloor?dsid=3504175fe000000000016be7&floor=14
{
”ok” :  true
}
```

4.7.16
------

getFloor

Read floor number where the device is installed.

**Synopsis**

```json
HTTP GET /json/device/getFloor
```

**Response**

```json
HTTP Status 200
```

floor the floor number where the device is installed.

**Sample**

```json
GET /json/device/getFloor?dsid=3504175fe000000000016be7
{
”ok” :  true ,
”result” :
{
”floor” : 14
}
}
```

4.7.17
------

getMaxMotionTime

Reads the maximum motion time configuration of a shade device.

**Synopsis**

```json
HTTP GET /json/device/getMaxMotionTime
```

Parameter

None

**Response**

```json
HTTP Status 200
```

result.supported boolean flag indicating if device supports this configuration

**Sample**

```json
GET /json/device/getMaxMotionTime?dsuid=3504175fe00000000000000000017bf600
{
”result” :
{
”supported” :  false ,
”value” : 0
},
”ok” :  true
}
```

4.7.18
------

setMaxMotionTime

Configures the maximum motion time of a shade device.

**Synopsis**

```json
HTTP GET /json/device/setMaxMotionTime
```

| Parameter | Description | Remarks |
|---|---|---|
| seconds | Maximummotiontimeinsecondswhere(0<seconds<655) | Mandatory |

Maximum motion time in seconds where (0 < seconds < 655) Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setMaxMotionTime?dsid=3504175fe000000000016be7&seconds=10
{
”ok” :  true
}
```

4.7.19
------

getOutputAfterImpulse

Reads configuration of an UMR device output after an impulse.

**Synopsis**

```json
HTTP GET /json/device/getOutputAfterImpulse
```

None

**Response**

```json
HTTP Status 200
```

result.output current output after impulse setting, can be ”on”, ”off” or ”retain”

**Sample**

```json
HTTP GET /json/device/getOutputAfterImpulse?dsuid=302ed89f43f0000000000ec00009478a00
{
”result” :
{
”output” :  ”retain”
},
”ok” :  true
}
```

4.7.20
------

setOutputAfterImpulse

Configures UMR device output after an impulse.

**Synopsis**

```json
HTTP GET /json/device/setOutputAfterImpulse
```

| Parameter | Description | Remarks |
|---|---|---|
| output | outputsetting: ”on”,”off”or”retain” | Mandatory |

output setting: ”on”, ”off” or ”retain” Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/getOutputAfterImpulse?dsuid=302ed89f43f0000000000ec00009478a00&output=off
{
”ok” :  true
}
```

4.7.21
------

setVisibility

Configure TNY device visibility. Not allowed for ”main” device.

**Synopsis**

```json
HTTP GET /json/device/setVisibility
```

| ok | true |
|---|---|

visibility setting: 0 or 1 Mandatory

**Response**

```json
HTTP Status 200
```

result.devices array of devices to be processed

**Sample**

```json
/json/device/setVisibility?dsuid=302ed89f43f00000000005400009f75d00&visibility=1
{
”result” :
{
”action” :  ”none”
},
”ok” :  true
}
```

4.7.22
------

setSupportedBasicScenes

Enables to provide a set of basic scenes supported by ventilation devices. Available scene numbers and

their descriptions are: 0 - ”Off”, 5 - ”Level1”, 17 - ”Level2”, 18 - ”Level3”, 19 - ”Level4”, 36 - ”Boost”.

**Synopsis**

```json
HTTP GET /json/device/setSupportedBasicScenes
```

| result.action result.devices | oneof”add”,”none”,”remove” arrayofdevicestobeprocessed |
|---|---|

Array of scene numbers Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
/json/device/setSupportedBasicScenes?dsuid=302ed89f43f00000000005400009f75d00&value=[0,5,17,18,19,36]
{
”ok” :  true
}
```

4.7.23
------

getSupportedBasicScenes

Reads out a set of basic scenes supported by ventilation device.

**Synopsis**

```json
HTTP GET /json/device/getSupportedBasicScenes
```

None

**Response**

```json
HTTP Status 200
```

value Array of supported basic scenes

**Sample**

```json
/json/device/getSupportedBasicScenes?dsuid=302ed89f43f00000000005400009f75d00
{
”result” :
{
”value” : [0,17,18]
},
”ok” :  true
}
```

4.7.24
------

setIgnoreOperationLock

Sets ignore operation lock on a device

**Synopsis**

```json
HTTP GET /json/device/setIgnoreOperationLock
```

| value | Arrayofsupportedbasicscenes |
|---|---|

lockStatus lock status, boolean: 1 = lock ignored, 0 = lock observed Mandatory (0 or 1)

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setIgnoreOperationLock?dsuid=302ed89f43f0000000000ec00009478a00&lockStatus=1
{
”ok” :  true
}
```

4.7.25
------

getIgnoreOperationLock

Gets ignore operation lock on a device

**Synopsis**

```json
HTTP GET /json/device/getIgnoreOperationLock
```

**Response**

```json
HTTP Status 200
```

ignoreOperationLock information whether ignore operation lock is set

**Sample**

```json
GET /json/device/getIgnoreOperationLock?dsuid=302ed89f43f0000000000ec00009478a00
{
”ok” :  true
”result” :
{
”ignoreOperationlock” :  false
}
}
```

4.8
---

Sensor

4.8.1
-----

Get Sensor Value

Ready a sensor measurement from a device.

**Synopsis**

```json
HTTP GET /json/device/getSensorValue
```

sensorIndex Numerical value, in the range from 0 to 14 Mandatory

**Response**

```json
HTTP Status 200
```

sensorIndex the index parameter from the request

sensorValue the actual measurement read from the device

**Sample**

```json
GET /json/device/getSensorValue?dsid=3504175fe000000000017ef3&sensorIndex=4
{
”ok” :  true ,
”result” : {
”sensorIndex” : 4,
”sensorValue” : 0
}
}
```

4.8.2
-----

Get Sensor Type

Ready the sensor type description from a device. For details about sensor types see the ds-basics

reference document.

**Synopsis**

```json
HTTP GET /json/device/getSensorType
```

sensorIndex Numerical value, in the range from 0 to 14 Mandatory

**Response**

```json
HTTP Status 200
```

sensorIndex the index parameter from the request

**Sample**

```json
GET /json/device/getSensorType?dsid=3504175fe000000000017ef3&sensorIndex=4
{
”ok” :  true ,
”result” : {
”sensorIndex” : 4,
”sensorType” : 6
}
}
```

4.8.3
-----

getSensorEventTableEntry

Reads the device event configuration for a given index. For details about the event table configuration

see the ds-basics reference document.

**Synopsis**

```json
HTTP GET /json/device/getSensorEventTableEntry
```

| Parameter | Description | Remarks |
|---|---|---|
| sensorIndex | Numericalvalue,intherangefrom0to14 | Mandatory |

eventIndex Index of the event configuration entry Mandatory

**Response**

```json
HTTP Status 200
```

sensorIndex Sensor index on which this entry operates

**Sample**

```json
GET /json/device/getSensorEventTableEntry?dsid=3504175fe00000000001540c&eventIndex=0
{
”ok” :  true ,
”result” : {
”eventIndex” : 0,
”eventName” :  ”” ,
”sensorIndex” : 2,
”test” : 2,
”action” : 0,
”value” : 35,
”hysteresis” : 0,
”validity” : 2
}
}
```

4.8.4
-----

setSensorEventTableEntry

Sets the device event configuration for a given index. For details about the event table configuration

see the ds-basics reference document.

**Synopsis**

```json
HTTP GET /json/device/setSensorEventTableEntry
```

sensorIndex Sensor index on which this entry operates Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setSensorEventTableEntry?dsid=3504175fe00000000001540c&eventIndex=0&eventName= ”TV￿turned￿on” &
sensorIndex=2&test=2&action=0&value=50&hysteresis=25&validity=2
{
”ok” :  true
}
```

4.9
---

Programming

4.9.1
-----

Set Programming Mode

Enabled or disabled the programming mode on a device.

**Synopsis**

```json
HTTP GET /json/device/setProgMode
```

| Parameter | Description | Remarks |
|---|---|---|
| mode | modevalue,eitherenabledordisabled | Mandatory |

mode value, either enabled or disabled Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setProgMode?dsid=3504175fe000000000017ef3&mode=disabled
{
”ok” :  true
}
```

4.9.2
-----

Add To Area

Modify the device scene table configuration and activate the area scene.

**Synopsis**

```json
HTTP GET /json/device/addToArea
```

| ok | true |
|---|---|

areaScene either the area-on or area-off scenes Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/addToArea?dsid=3504175fe000000000017ef3&areaScene=7
{
”ok” :  true
}
```

4.9.3
-----

Remove From Area

Modify the device scene table configuration and deactivate the area scene.

**Synopsis**

```json
HTTP GET /json/device/removeFromArea
```

| Parameter | Description | Remarks |
|---|---|---|
| areaScene | eitherthearea-onorarea-offscenes | Mandatory |

areaScene either the area-on or area-off scenes Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/removeFromArea?dsid=3504175fe000000000017ef3&areaScene=7
{
”ok” :  true
}
```

4.9.4
-----

Get Transmission Quality

Sends test commands to a device to evaluate the actual transmission quality.

**Synopsis**

```json
HTTP GET /json/device/getTransmissionQuality
```

None

**Response**

```json
HTTP Status 200
```

downstream a numerical value in the range 0 to 6, 0 meaning best quality

**Sample**

```json
GET /json/device/getTransmissionQuality?dsid=3504175fe000000000017ef3
{
”ok” :  true ,
”result” : {
”upstream” : 61,
”downstream” : 0
}
}
```

4.10
----

Heating and valve actuators

4.10.1
------

setHeatingGroup

Sets the standard color group of a heating actuator. Some actuators support operation with differ-

ent connected hardware equipment, therefore the terminal blocks support operation in different zone

groups, for example in heating, cooling or ventilation.

**Synopsis**

```json
HTTP GET /json/device/setHeatingGroup
```

| Parameter | Description | Remarks |
|---|---|---|
| groupID | NewgroupId | Mandatory |

New group Id Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setHeatingGroup?dsuid=3504175fe00000000000000000016be700&groupID=48
{
”ok” :  true
}
```

4.10.2
------

getValvePwmState

Reads the device status of a valve PWM actuator.

**Synopsis**

```json
HTTP GET /json/device/getValvePwmState
```

None

**Response**

```json
HTTP Status 200
```

pwmPriorityMode Current operating state value

**Sample**

```json
GET /json/device/getValvePwmState?dsuid=3504175fe00000000000000000016be700
{
”ok” :  true ,
”result” :
{
”pwmValue” : 60,
”pwmPriorityMode” : 0
}
}
```

4.10.3
------

getValvePwmMode

Reads the device configuration of a valve PWM actuator.

**Synopsis**

```json
HTTP GET /json/device/getValvePwmMode
```

None

**Response**

```json
HTTP Status 200
```

pwmPeriod Length of PWM period in seconds

**Sample**

```json
GET /json/device/getValvePwmMode?dsuid=3504175fe00000000000000000016be700
{
”ok” :  true ,
”result” :
{
”pwmPeriod” : 900,
”pwmMinX” : 0,
”pwmMaxX” : 10,
”pwmMinY” : 12,
”pwmMaxY” : 28,
”pwmOffset” :  − 20
}
}
```

4.10.4
------

setValvePwmMode

Sets the device configuration of a valve PWM actuator.

**Synopsis**

```json
HTTP GET /json/device/setValvePwmMode
```

pwmPeriod Length of PWM period in seconds, 0 .. 64k

Minimum set point or threshold, 0 .. 255

Maximum set point or threshold, 0 .. 255

Minimum output value at min set point, 0 .. 255

Maximum output value at max set point, 0 .. 255 Optional

Set point offset, -128 .. 127

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setValvePwmMode?dsuid=3504175fe00000000000000000016be700&pwmMaxX=80&pwmOffset= − 20
{
”ok” :  true
}
```

4.10.5
------

getValveControlMode

Reads the device configuration of a valve PWM actuator.

**Synopsis**

```json
HTTP GET /json/device/getValveControlMode
```

Parameter

None

**Response**

```json
HTTP Status 200
```

ctrlClipMaxHigher PWM value over maximum control value to 100 percent

**Sample**

```json
GET /json/device/getValveControlMode?dsuid=3504175fe00000000000000000016be700
{
”ok” :  true ,
”result” :
{
”ctrlNONC” :  true ,
”ctrlClipMaxHigher” :  false ,
”ctrlClipMinLower” :  false ,
”ctrlClipMinZero” :  false
}
}
```

4.10.6
------

setValveControlMode

Sets the device configuration of a valve PWM actuator.

**Synopsis**

```json
HTTP GET /json/device/setValveControlMode
```

ctrlClipMaxHigher PWM value over maximum forces control value

Configure normally open or normally closed

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setValveControlMode?dsuid=3504175fe00000000000000000016be700&ctrlNONC= false
{
”ok” :  true
}
```

4.10.7
------

getValveTimerMode

Reads the timer device configuration settings of a valve PWM actuator.

**Synopsis**

```json
HTTP GET /json/device/getValveTimerMode
```

None

**Response**

```json
HTTP Status 200
```

valveProtectionTimer Duration of the valve protection period in seconds

**Sample**

```json
GET /json/device/getValveTimerMode?dsuid=3504175fe00000000000000000016be700
{
”ok” :  true ,
”result” :
{
”valveProtectionTimer” : 0,
”emergencyValue” : 75,
”emergencyTimer” : 7200
```

```json
}
}
```

4.10.8
------

setValveTimerMode

Sets the timer device configuration of a valve PWM actuator.

**Synopsis**

```json
HTTP GET /json/device/setValveTimerMode
```

valveProtectionTimer Duration of the valve protection period in sec-

Fixed output value in percent in emergency

Duration in seconds until emergency mode is

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setValveTimerMode?dsuid=3504175fe00000000000000000016be700&valveProtectionTimer=600
{
”ok” :  true
}
```

4.11
----

Single Device Info

The  Single Device Info  section refers to the device description data that is available only for selected

devices. Please read the  Device Description  section of the system interfaces documentation.

If the device is not a Single Device with device description data the following error message will be

returned:

```json
{
”ok” :  false ,
”message” :  ”Device￿does￿not￿support￿action￿configuration”
}
```

4.11.1
------

Get Info Static

Returns the static device description data of a device. This data is available in a database on the

digitalSTROM-Server and is not fetched from the device itself.

**Synopsis**

```json
HTTP GET /json/device/getInfoStatic
```

| Parameter | Description | Remarks |
|---|---|---|
| lang | LocaleCodebasedonLanguage_Regionpattern,e.g. en_EN,de_DE | Mandatory |

Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory

**Response**

```json
HTTP Status 200
```

propertyDescriptions list of property description objects

```json
”spec” : {
”descriptionId” : {
”title” :  ”Translated￿title￿for￿key￿descriptionId” ,
”value” :  ”Value￿of￿the￿key￿descriptionId” ,
”tags” :  ”string￿value,￿semi − colon￿separated￿list￿of￿attributes”
}
}
```

Common descriptionId’s are:

Instance ID of the hardware, e.g. S/N, MAC Address, SGTIN

”hardwareModelGuid” Model ID of the hardware, e.g. GTIN of the Native Device

dS Class/Profile Name, e.g. ”Water Kettle”, ”Dishwasher”

```json
”stateTechnicalName” : {
”title” :  ”Translated￿title￿for￿this￿state￿object” ,
”options” : { list of  ”OptionId” :  ”OptionValue”  pairs }
”tags” :  ”optional￿string￿value,￿semi − colon￿separated￿list￿of￿attributes”
}
```

Example:

```json
”operation” : {
”title” :  ”Betriebszustand” ,
”options” : {
”idle” :  ”Bereitschaft” ,
”active” :  ”Aktiv” ,
”error” :  ”Fehler”
},
”tags” :  null
}
```

eventDescriptions

```json
”eventTechnicalName” : {
”title” :  ”Translated￿title￿for￿this￿event￿object” ,
”accessLevel” : optional enum of
”normal”  ( default )  // Shown everywhere
”advanced”  // Shown within the UI if the  ‘ show more  ’ events option was selected
”service”  // Not shown within the  ’ UIs at all but might be used by bots and other services.
}
```

propertyDescriptions

```json
”parameterTechnicalName” : {
”title:” ”Translated￿name￿of￿this￿parameter” ,
”type” :  ”data￿type￿of￿the￿parameter￿value:￿numeric,￿enumeration,￿string” ,
”tags” :  ”string￿value,￿semi − colon￿separated￿list￿of￿attributes”
}
```

Additional optional fields for type  numeric :

resolution minimum step size

unit string, http://www.ebyte.it/library/educards/siunits/TablesOfSiUnitsAndPrefixes.html, e.g. ”sec

Additional optional fields for type  enumeration :

options json object with a list of ”OptionId”: ”OptionValue” pairs

default a default value of the property

Additional optional fields for type  text :

default a default value of the property

Following fields are defined for the  tags  attribute:

readonly parameter value can only be read and not written

overview state or property shall be shown on the overview tab in the UI, order/position can be given with ”:num

The ”type” field string might have a postfix that indicates the characteristic of the value. This can be

used for rendering the data field in user interfaces.

Following postfix descriptions are defined for the  type  attribute:

numeric.timeOfDay hh:mm or am/pm depending on the region setting, does not include time zone

numeric.duration

hh:mm:ss or hh:mm, depending on unit and resolution

numeric.boolean

true/false, displayed as checkbox

Example:

```json
”waterhardness” : {
”title” :  ”Wasserhärte” ,
”type” :  ”numeric” ,
”min” : 0,
”max” : 6,
”resolution” : 0.1,
”default” : 2.1,
”tags” :  ”readonly”
}
```

sensorDescriptions

```json
”sensorTechnicalName” : {
”title:” ”Translated￿name￿of￿this￿measurement” ,
”type” :  ”data￿type￿of￿the￿measurement￿value:￿numeric,￿enumeration,￿string” ,
”tags” :  ”string￿value,￿semi − colon￿separated￿list￿of￿attributes”
}
```

The sensor object is represented by the same extended fields then property objects, depending on the

”type” field.

Additional mandatory fields for sensor objects are:

dsIndex device index of the source, necessary to address in queries

actionDescriptions

```json
”actionDescriptions” : {
”actionId1” : {
”title” :  ”Translated￿label￿for￿actionId1” ,
”params” : list of {propertyDescriptions}
},
....,
”actionIdN” : {
”title” :  ”Translated￿label￿for￿actionIdN” ,
”params” : list of {propertyDescriptions}
}
},
```

standardActions

```json
”standardActions” : {
”std.Action1”  : {
”action” :  ”reference￿to￿the￿base￿action￿description” ,
”title” :  ”Translated￿name￿for￿std.Action1” ,
”params” : { list of  ”ParameterName” : ParameterValue, ...}
},
```

```json
...,
”std.ActionN”  : {
”action” :  ”reference￿to￿the￿base￿action￿description” ,
”title” :  ”Translated￿name￿for￿std.ActionN” ,
”params” : { list of  ”ParameterName” : ParameterValue, ...}
}
}
```

**Sample**

```json
GET /json/device/getInfoStatic?dsuid=687ba4e345e75bd58093bf119f8a6c6700&lang=de_DE
{
”result” : {
”spec” : {
”dsDeviceGTIN” : {
”title” :  ”dS￿Device￿GTIN” ,
”tags” :  ”settings:5” ,
”value” :  ”7640156791945”
},
”hardwareGuid” : {
”title” :  ”Artikel￿Kennzeichnung” ,
”tags” :  ”settings:4” ,
”value” :  ”MAC￿5C:CF:7F:11:F8:B8”
},
”hardwareModelGuid” : {
”title” :  ”Produkt￿Kennzeichnung” ,
”tags” :  ”invisible” ,
”value” :  ”smartermodel:ikettle2”
},
”model” : {
”title” :  ”Modell” ,
”tags” :  ”overview:2;settings:2” ,
”value” :  ”iKettle￿2”
},
”modelVersion” : {
”title” :  ”Modellvariante” ,
”tags” :  ”invisible” ,
”value” :  ”19”
},
”name” : {
”title” :  ”Name” ,
”tags” :  ”overview:1;settings:1” ,
”value” :  ”Wasserkocher”
},
”notes” : {
”title” :  ”Bemerkungen” ,
”tags” :  ”overview:4” ,
”value” :  ”Bitte￿prüfen￿Sie￿mit￿der￿’Smarter’￿Smartphone￿App,￿ob￿die￿iKettle￿Firmware￿auf￿dem￿aktuellsten￿Stand￿ist!”
},
”vendorId” : {
”title” :  ”Hersteller￿Kennung” ,
”tags” :  ”invisible” ,
”value” :  ”vendorname:Smarter￿Applications￿Ltd.”
},
”vendorName” : {
”title” :  ”Hersteller” ,
”tags” :  ”overview:3;settings:3” ,
”value” :  ”Smarter￿Applications￿Ltd.”
},
”class” : {
”title” :  ”Geräteklasse” ,
”tags” :  ”invisible” ,
”value” :  ””
},
”classVersion” : {
”title” :  ”Geräteklassen￿Version” ,
”tags” :  ”invisible” ,
”value” :  ””
}
},
”stateDescriptions” : {
”operation” : {
”title” :  ”Betriebsmodus” ,
”tags” :  ”overview” ,
”options” : {
```

```json
”cooldown” :  ”Abkühlen” ,
”heating” :  ”Aufheizen” ,
”keepwarm” :  ”Warmhalten” ,
”ready” :  ”Bereit” ,
”removed” :  ”Abgehoben”
}
}
},
”eventDescriptions” : {
”KettleAttached” : {
”title” :  ”Kocher￿aufgesetzt”
},
”BoilingStarted” : {
”title” :  ”Aufheizen￿gestartet”
},
”KeepWarm” : {
”title” :  ”Warmhalten￿gestartet”
},
”BabycoolingStarted” : {
”title” :  ”Aufheizen￿beendet,￿￿auf￿Zieltemperatur￿abkühlen”
},
”BoilingFinished” : {
”title” :  ”Aufheizen￿beendet”
},
”KettleReleased” : {
”title” :  ”Kocher￿abgehoben”
},
”BabycoolingFinished” : {
”title” :  ”Abkühlen￿beendet,￿Zieltemperatur￿erreichet”
},
”KeepWarmFinished” : {
”title” :  ”Warmhalten￿beendet”
},
”BoilingAborted” : {
”title” :  ”Aufheizen￿abgebrochen,￿Taste￿betätigt”
},
”BabycoolingAborted” : {
”title” :  ”Abkühlen￿abgebrochen,￿Taste￿betätigt”
},
”KeepwarmAborted” : {
”title” :  ”Warmhalten￿abgebrochen,￿Taste￿betätigt”
},
”KeepWarmAfterBoiling” : {
”title” :  ”Aufheizen￿beendet,￿warmhalten”
},
”KeepWarmAfterBabycooling” : {
”title” :  ”Abkühlen￿beendet,￿warmhalten”
},
”BoilingAbortedAndKettleReleased” : {
”title” :  ”Aufheizen￿abgebrochen,￿Kocher￿abgehoben”
},
”BabyCoolingAbortedAndKettleReleased” : {
”title” :  ”Abkühlen￿abgebrochen,￿Kocher￿abgehoben”
},
”KeepWarmAbortedAndKettleReleased” : {
”title” :  ”Warmhalten￿abgebrochen,￿Kocher￿abgehoben”
}
},
”propertyDescriptions” : {
”currentTemperature” : {
”title” :  ”Wassertemperatur” ,
”tags” :  ”readonly;overview” ,
”type” :  ”numeric” ,
”min” :  ”0” ,
”max” :  ”100” ,
”resolution” :  ”1” ,
”siunit” :  ”celsius” ,
”default” :  ”0”
},
”waterLevel” : {
”title” :  ”Füllstand” ,
”tags” :  ”readonly;overview” ,
”type” :  ”numeric” ,
”min” :  ”0” ,
”max” :  ”2.0” ,
”resolution” :  ”0.2” ,
”siunit” :  ”liter” ,
”default” :  ”0”
```

```json
},
”defaulttemperature” : {
”title” :  ”Temperatur￿Aufheizen” ,
”tags” :  ”settings” ,
”type” :  ”numeric” ,
”min” :  ”0” ,
”max” :  ”100” ,
”resolution” :  ”1” ,
”siunit” :  ”celsius” ,
”default” :  ”100”
},
”defaultcooldowntemperature” : {
”title” :  ”Temperatur￿Abkochen￿und￿Abkühlen” ,
”tags” :  ”invisible” ,
”type” :  ”numeric” ,
”min” :  ”0” ,
”max” :  ”100” ,
”resolution” :  ”1” ,
”siunit” :  ”celsius” ,
”default” :  ”80”
},
”defaultkeepwarmtime” : {
”title” :  ”Warmhaltezeit” ,
”tags” :  ”settings” ,
”type” :  ”numeric” ,
”min” :  ”0” ,
”max” :  ”30” ,
”resolution” :  ”1” ,
”siunit” :  ”min” ,
”default” :  ”15”
}
},
”sensorDescriptions” : {
”waterQuantity” : {
”title” :  ”Wassermenge” ,
”tags” :  ”readonly;overview” ,
”type” :  ”numeric” ,
”min” :  ”0” ,
”max” :  ”8” ,
”resolution” :  ”0.1” ,
”siunit” :  ”liter” ,
”dsType” : 68,
”dsIndex” : 3
}
},
”actionDescriptions” : {
”boilandcooldown” : {
”title” :  ”Abkochen￿und￿abkühlen” ,
”params” : {
”keepwarmtime” : {
”title” :  ”Warmhaltedauer” ,
”tags” :  ”” ,
”type” :  ”numeric” ,
”min” :  ”0” ,
”max” :  ”30” ,
”resolution” :  ”1” ,
”siunit” :  ”min” ,
”default” :  ”30”
},
”temperature” : {
”title” :  ”Zieltemperatur” ,
”tags” :  ”” ,
”type” :  ”numeric” ,
”min” :  ”20” ,
”max” :  ”100” ,
”resolution” :  ”1” ,
”siunit” :  ”celsius” ,
”default” :  ”50”
}
}
},
”heat” : {
”title” :  ”Aufheizen” ,
”params” : {
”keepwarmtime” : {
”title” :  ”Warmhaltedauer” ,
”tags” :  ”” ,
”type” :  ”numeric” ,
```

```json
”min” :  ”0” ,
”max” :  ”30” ,
”resolution” :  ”1” ,
”siunit” :  ”min” ,
”default” :  ”30”
},
”temperature” : {
”title” :  ”Zieltemperatur” ,
”tags” :  ”” ,
”type” :  ”numeric” ,
”min” :  ”20” ,
”max” :  ”100” ,
”resolution” :  ”1” ,
”siunit” :  ”celsius” ,
”default” :  ”100”
}
}
},
”stop” : {
”title” :  ”Abschalten” ,
”params” : {}
}
},
”standardActions” : {
”std.boilandcooldown” : {
”title” :  ”Abkochen￿und￿abkühlen” ,
”action” :  ”boilandcooldown” ,
”params” : {
”temperature” :  ”40”
}
},
”std.heat” : {
”title” :  ”Aufheizen” ,
”action” :  ”heat” ,
”params” : {}
},
”std.stop” : {
”title” :  ”Abschalten” ,
”action” :  ”stop” ,
”params” : {}
}
}
},
”ok” :  true
}
```

4.11.2
------

Get Info Operational

Returns the current value for states and properties.

**Synopsis**

```json
HTTP GET /json/device/getInfoOperational
```

| Parameter | Description | Remarks |
|---|---|---|
| lang | LocaleCodebasedonLanguage_Regionpattern,e.g. en_EN,de_DE | Mandatory |

Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory

**Response**

```json
HTTP Status 200
```

properties list of property value objects

States and Property names are corresponding to the response of the static descriptions. The static

response contains translations and other meta information.

states

```json
”states” : {
”stateTechnialName1” : {
”value” :  ”stateOptionValue”
},
...,
”stateTechnicalNameN” : {
”value” :  ”stateOptionValue”
}
},
```

properties

```json
”properties” : {
”propertyTechnialName1” : {
”value” : number
},
...,
”propertyTechnicalNameN” : {
”value” : number
}
},
```

measurements

```json
”sensors” : {
”sensorTechnialName1” : {
”value” : number
},
...,
”sensorTechnicalNameN” : {
”value” : number
}
},
```

**Sample**

```json
GET /json/device/getInfoOperational?dsuid=687ba4e345e75bd58093bf119f8a6c6700&lang=de_DE
{
”result” : {
”states” : {
”operation” : {
”age” : 1.756285,
”changed” : 45452.848575,
”value” :  ”ready”
}
},
”properties” : {
”currentTemperature” : 25,
”defaultkeepwarmtime” : 30,
”defaulttemperature” : 100,
”waterLevel” : 1.8620689655172413
},
”sensors” : {
”waterQuantity” : 8.6
}
},
”ok” :  true
}
```

4.11.3
------

Get Info Custom

Returns the custom actions defined by the user or define. The custom actions are configurable and are

available in addition to the standard actions.

**Synopsis**

```json
HTTP GET /json/device/getInfoCustom
```

| Parameter | Description | Remarks |
|---|---|---|
| lang | LocaleCodebasedonLanguage_Regionpattern,e.g. en_EN,de_DE | Mandatory |

Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory

**Response**

```json
HTTP Status 200
```

customActions list of action description objects

The custom action name is given by the user. Each custom action is based on a defined and known

actionDescription of the device.

```json
”customActions” : {
”custom.753151” : {
”action” :  ”reference￿to￿the￿base￿action￿description” ,
”title” :  ”User￿given￿name￿for￿custom.753151” ,
”params” : { list of  ”ParameterName” : ParameterValue, ...}
},
....,
”custom.143937” : {
”action” :  ”reference￿to￿the￿base￿action￿description” ,
”title” :  ”User￿given￿name￿for￿custom.143937” ,
”params” : { list of  ”ParameterName” : ParameterValue, ...}
}
}
```

**Sample**

```json
GET /json/device/getInfoCustom?dsuid=687ba4e345e75bd58093bf119f8a6c6700&lang=de_DE
{
”result” : {
”customActions” : {
”custom.582620628227b” : {
”action” :  ”std.boilandcooldown” ,
”params” : {
”keepwarmtime” : 30,
”temperature” : 70
},
”title” :  ”Früchtetee”
},
”custom.5826208698525” : {
”action” :  ”std.heat” ,
”params” : {
”keepwarmtime” : 0,
”temperature” : 42
},
”title” :  ”Lauwarmes￿Wasser”
},
”custom.582620a92e329” : {
”action” :  ”std.heat” ,
```

```json
”params” : {
”keepwarmtime” : 15,
”temperature” : 66
},
”title” :  ”Spülwasser￿aufwärmen”
},
”custom.58404582ef972” : {
”action” :  ”std.boilandcooldown” ,
”params” : {
”keepwarmtime” : 0,
”temperature” : 100
},
”title” :  ”Wasser￿abkochen”
}
}
},
”ok” :  true
}
```

4.11.4
------

Get Info

getInfo is a method that combines static, operational and custom information in one call. With the given

filter  parameter the caller can select which response fields he likes to have in the response.

**Synopsis**

```json
HTTP GET /json/device/getInfo
```

| Parameter | Description | Remarks |
|---|---|---|
| lang | LocaleCodebasedonLanguage_Regionpattern,e.g. en_EN,de_DE | Mandatory |
| filter | stringwithacommaseparatedlistofresponseobjects | Optional |

Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory

The filter parameter accepts the following options: spec, standardActions, customActions, stateDesc,

propertyDesc, sensorDesc, actionDesc, eventDesc, operational.

If filter parameter is omitted or empty the full set of response objects is returned.

**Response**

```json
HTTP Status 200
```

The response is a combination of the getInfoStatic, getInfoCustom and getInfoOperational response.

Please refer to the descriptions above.

4.11.5
------

Set Property

This method allows to change property values that are part of a getInfo Property Description.

**Synopsis**

```json
HTTP GET /json/device/setProperty
```

| Parameter | Description | Remarks |
|---|---|---|
| dsuid | dSUIDofthedevice | Mandatory |
| id | propertyidentifier | Mandatory |
| value | newvalueoftheproperty | Mandatory |

new value of the property Mandatory

The  id  parameter corresponds to the property technical name from getInfo response.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setProperty?dsuid=687ba4e345e75bd58093bf119f8a6c6700&id=defaultkeepwarmtime&value=90.5
{
”ok” :  true
}
```

4.11.6
------

Set Custom Action

This method allows to create or replace custom actions of a device.

**Synopsis**

```json
HTTP GET /json/device/setCustomAction
```

| ok | true |
|---|---|

json object with a list of property values: ”PropertyName”: ”PropertyValue” , ...

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setCustomAction?dsuid=687ba4e345e75bd58093bf119f8a6c6700&id=custom.123456&title=Lauwarmes Wasser&
action=std.heat&params={ ”temperature” :40}
{
”ok” :  true
}
```

4.11.7
------

Call Action

Excutes the action  id  on a device.

**Synopsis**

```json
HTTP GET /json/device/callAction
```

| Parameter | Description | Remarks |
|---|---|---|
| id | standardorcustomactionidentifier | Mandatory |

standard or custom action identifier Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/callAction?dsuid=df6aa5bba4db5540c0fe55e3eb088be900&id=std.stop
{
”ok” : true
}
```

4.11.8
------

Get Apartment Scenes

Retrieves the list of device values for all supported apartment scenes (sceneID 64 and above).

**Synopsis**

```json
HTTP GET /json/device/getSceneValue
```

None

**Response**

```json
HTTP Status 200
```

result.scenes a list of json objects per sceneID

**Sample**

```json
GET /json/device/getApartmentScenes?dsuid=df6aa5bba4db5540c0fe55e3eb088be900
”result” : {
”scenes” : {
”64” : {
”channels” :  null ,
”command” :  ”std.stop” ,
”dontCare” :  false ,
”effect” : 1,
”ignoreLocalPriority” :  true
},
....,
”92” : {
”channels” :  null ,
```

```json
”command” :  ”std.stop” ,
”dontCare” :  false ,
”effect” : 1,
”ignoreLocalPriority” :  true
}
}
},
”ok” :  true
}
```

4.12
----

Test presence

4.12.1
------

testPresence

Tests the device presence.

**Synopsis**

```json
HTTP GET /json/device/testPresence
```

Parameter

None

**Response**

```json
HTTP Status 200
```

result.active bool - is the device active / inactive?

**Sample**

```json
GET /json/device/getName?dsid=3504175fe000000000017ef3
{
”ok” : true ,
”result”  :
{
”active”  :  true
}
}
```

4.13
----

Button Usage

Most devices have a button input, but not all of them are connected to push-buttons. A user interface

may only display button inputs which were used before, ignoring all the terminal blocks where no but-

tons are connected to. A button is ”used” if it got pushed before, ”auto_unused” if it got not pushed yet,

or ”manual_unused” if a user set it into this state.

4.13.1
------

getButtonUsage

Returns information about button input usage of the device, only supported on devices that have button

inputs.

**Synopsis**

```json
HTTP GET /json/device/getButtonUsage
```

Parameter

None

**Response**

```json
HTTP Status 200
```

result.buttonUsage button usage setting (used, auto_unused, manual_unused, unsupported)

**Sample**

```json
GET /json/device/getButtonUsage?dsuid=3504175fe0000000000000016c4f00
{
”ok” : true ,
”result”  :
{
”buttonUsage”  :  ”used”
}
}
```

4.13.2
------

setButtonUsage

Sets the button usage state, only supports devices that have buttons.

**Synopsis**

```json
HTTP GET /json/device/setButtonUsage
```

| result.buttonUsage | buttonusagesetting(used,auto_unused,manual_unused,unsupported) |
|---|---|

usage identifier (used, auto_unused, manual_unused) Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/device/setButtonUsage?dsuid=3504175fe0000000000000016c4f00&usage=manual_unused
{
”ok” : true
}
```
