4 Device
========

4
Device
4.1
Common
Every /json/device/ function uses a common selection scheme for the device to which the command
refers to. Either the parameter ”dsid” or ”name” must be given to identify the device.
Parameter Description
Remarks
dsid
Device dSID String Optional
name
Device Name
Optional
category
Request Category
Optional
A missing device identifier result in the following error message to be returned.
{ ”ok”: false, ”message”: ”Need￿parameter￿name￿or￿dsid￿to￿identify￿device” }
If a device identifier does not match any actually known device in the installation the following error
message is returned.
{ ”ok”: false, ”message”: ”Could￿not￿find￿device￿named￿’Wandlampe￿am￿Eingang’” }
The category parameter has an influence on how particular requests are treated, the goal is to prevent
scene calls from automated scripts in certain situations. Currently supported categories are:
• manual
• timer
• algoirthm
A missing category parameter is currently treated as manual category, this compatibility will be re-
moved in release 1.8.
4.2
Name
4.2.1
getName
Returns the user defined name of a device.
Synopsis
HTTP GET /json/device/getName
Parameter
None
Response
HTTP Status 200
result.name identifier string for the device
Sample
GET /json/device/getName?dsid=3504175fe000000000017ef3
{
”ok”:true,
”result” :
{
”name” : ”App−Taster”
62

}
}
4.2.2
setName
Sets the device name.
Synopsis
HTTP GET /json/device/setName
Parameter Description
Remarks
newName
identifier string for the device Mandatory
Parameter
Response
HTTP Status 200
ok true
Sample
GET /json/device/setName?id=3504175fe000000000017ef3&newName=”Wohnen”
{
”ok”:true
}
4.2.3
getSpec
Retrieves device and product information.
Synopsis
HTTP GET /json/device/getSpec
Parameter
None
Response
HTTP Status 200
result.functionID Function ID of the device
result.productID Product ID of the device
result.revisionID Revision ID of the device
Sample
GET /json/device/getName?dsid=3504175fe000000000017ef3
{
”ok”: true,
”result”: {
”functionID”: 33027,
”productID”: 1224,
63

”revisionID”: 834
}
}
4.3
First seen
4.3.1
getFirstSeen
Returns the timestamp when the device was registered.
Synopsis
HTTP GET /json/device/getFirstSeen
Parameter
None
Response
HTTP Status 200
result.time ISO8601 time when device was registered
Sample
GET /json/device/getFirstSeen?dsid=3504175fe000000000017ef3
{
”result” :
{
”time” : ”2010−10−01T10:42:13Z”
},
”ok”:true
}
4.4
Groups
4.4.1
getGroups
Returns a list of groups the device is registered in.
Synopsis
HTTP GET /json/device/getGroups
Parameter
None
Response
HTTP Status 200
result.groups array of groups of the device
Sample
64

GET /json/device/getGroups
{
”ok”: true,
”result”: {
”groups”: [
{
”id”: 3,
”name”: ”blue”
},
{
”id”: 8,
”name”: ”black”
}
]
}
}
4.5
Scene
4.5.1
callScene
Excutes the scene sceneNumber on a devices.
Synopsis
HTTP GET /json/device/callScene
Parameter
Parameter
Description
Remarks
sceneNumber Numerical value
Mandatory
force
Boolean value, if set a forced scene call is issued Optional
Response
HTTP Status 200
ok true
Sample
GET /json/device/callScene?dsid=3504175fe000000000017ef3&sceneNumber=13
{
”ok”:true
}
4.5.2
saveScene
Tells the device to store the current output values as a default for the scene sceneNumber.
Synopsis
HTTP GET /json/device/saveScene
Parameter
Parameter
Description
Remarks
sceneNumber Numerical value Mandatory
65

Response
HTTP Status 200
ok true
Sample
GET /json/device/saveScene?dsid=3504175fe000000000017ef3&sceneNumber=5
{
”ok”:true
}
4.5.3
undoScene
Tells devices to restore the output values to the previous state if the current scene matches the sce-
neNumber.
Synopsis
HTTP GET /json/device/undoScene
Parameter
Parameter
Description
Remarks
sceneNumber Numerical value Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/undoScene?dsid=3504175fe000000000017ef3&sceneNumber=18
{
”ok”:true
}
4.5.4
turnOn
Tells devices to execute the scene MAX.
Synopsis
HTTP GET /json/device/turnOn
Parameter
None
Response
HTTP Status 200
ok true
66

Sample
GET /json/device/turnOn?dsid=3504175fe000000000017ef3
{
”ok”:true
}
4.5.5
turnOff
Tells devices to execute the scene MIN.
Synopsis
HTTP GET /json/device/turnOff
Parameter
None
Response
HTTP Status 200
ok true
Sample
GET /json/device/turnOff?dsid=3504175fe000000000017ef3
{
”ok”:true
}
4.5.6
increaseValue
Tells devices to execute the scene INC.
Synopsis
HTTP GET /json/device/increaseValue
Parameter
None
Response
HTTP Status 200
ok true
Sample
GET /json/device/increaseValue?dsid=3504175fe000000000017ef3
{
”ok”:true
}
4.5.7
decreaseValue
Tells devices to execute the scene DEC.
67

Synopsis
HTTP GET /json/device/decreaseValue
Parameter
None
Response
HTTP Status 200
ok true
Sample
GET /json/device/decreaseValue?dsid=3504175fe000000000017ef3
{
”ok”:true
}
4.6
Value
4.6.1
Set Value
Set the primary output value of a device to a given value.
Notice Setting output values directly bypasses the group state machine and is unrecommended.
Synopsis
HTTP GET /json/device/setValue
Parameter
Parameter Description
Remarks
value
Numerical 8 bit value, in the range from 0 to 255 Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setValue?dsid=3504175fe000000000017ef3&value=127
{
”ok”:true
}
68

4.6.2
Set Output Value
Set a output channel value of a device to a given value. The available output parameter ranges and
channels depend on the feature of the hardware components.
Notice Setting output values directly bypasses the group state machine and is unrecommended.
Synopsis
HTTP GET /json/device/setOutputValue
Parameter
Parameter Description
Remarks
value
Numerical Value
Mandatory
offset
Output Data Offset Mandatory
offset Description
Product
2
shadeOpeningAngleOutside, 16 Bit
GR-KL220, GR-HKL230
Response
HTTP Status 200
ok true
Sample
GET /json/device/setOutputValue?dsid=3504175fe000000000017ef3&offset=2&value=5345
{
”ok”:true
}
4.6.3
Get Output Value
Get the current output channel status of a device. The available output channels depend on the feature
of the hardware components.
Notice Getting output values directly from the device takes a noticeable amount of time. This re-
quest is subject of limitations in the systems certification rules.
Synopsis
HTTP GET /json/device/getOutputValue
Parameter
Parameter Description
Remarks
offset
Output Data Offset Mandatory
type
Named Parameter Mandatory
69

Either offset or type parameter has to be given.
type
Description
Product
position
Target position, 16 Bit
GR-KL200, GR-KL210,
GR-KL220, GR-KL230
angle
Target angle position, 8 Bit
GR-KL220, GR-KL230
positionCurrent
Current position when active, 16 Bit
GR-KL200, GR-KL210,
GR-KL220, GR-KL230
pwmValue
Current PWM value, 8 Bit
BL-KM200,
BL-
SDS200
pwmPriorityMode
Current PWM priority value
BL-KM200,
BL-
SDS200
currentOutputMode output operation mode when automatic selec-
tion is active
GE-KM300,
GE-
TKM300
Response
HTTP Status 200
result.offset the given offset from the request
result.value Numerical value of the selected output channel queried from the device
Sample
GET /json/device/getOutputValue?dsid=3504175fe000000000017ef3&offset=0
{
”ok”: true, ”result”: { ”offset”: 0, ”value”: 5345 }
}
4.6.4
Get Scene Value
Retrieves the device value of the given scene.
Synopsis
HTTP GET /json/device/getSceneValue
Parameter
Parameter Description
Remarks
sceneID
Numerical value Mandatory
Response
HTTP Status 200
result.value
numerical output channel value of the device
result.angle
if available, angle value of the device
result.scenes if available, a json object with a command field that is either a standard or custom device action
70

Sample
GET /json/device/getSceneValue?dsuid=3504175fe0000000000000017ef300&sceneID=72
{
”ok”:true,”result”:{”value”:65535,”angle”:255}
}
GET /json/device/getSceneValue?dsuid=687ba4e345e75bd58093bf119f8a6c6700&sceneID=72
{
”ok”:true,”result”:{”value”:0}
}
GET /json/device/getSceneValue?dsuid=df6aa5bba4db5540c0fe55e3eb088be900&sceneID=72
”result”: {
”scenes”: {
”72”: {
”channels”: null,
”command”: ”std.stop”,
”dontCare”: false,
”effect”: 1,
”ignoreLocalPriority”: true
}
}
},
”ok”: true
}
4.6.5
Set Scene Value
Retrieves the device value of the given scene.
Synopsis
HTTP GET /json/device/setSceneValue
Parameter
Parameter Description
Remarks
sceneID
Numerical value
Mandatory
value
Numerical value
Mandatory
angle
Numerical value, if applicable Optional
command
String value, if applicable
Optional
Response
HTTP Status 200
ok true
Sample
GET /json/device/setSceneValue?dsuid=3504175fe0000000000000016c4f00&sceneID=72&value=26987
{
”ok”:true
}
GET /json/device/setSceneValue?dsuid=687ba4e345e75bd58093bf119f8a6c6700&sceneID=81&value=100&command=boilandcooldown
{
”ok”:true
}
71

4.6.6
Get Scene Mode
Reads the device configuration flags for a given sceneID. For details about the scene configuration see
the ds-basics reference document.
Synopsis
HTTP GET /json/device/getSceneMode
Parameter
Parameter Description
Remarks
sceneID
Scene number for which the configuration is requested Mandatory
Response
HTTP Status 200
sceneID
Scene number which has been requested
dontCare
Don’t Care Flag
localPrio
Local Prio Flag
specialMode
Special Mode Flag
flashMode
Flashing Mode Flag
ledconIndex
Index of the LED configuration register
dimmTimeIndex Index of the transition configuration register
Sample
GET /json/device/getSceneMode?dsid=3504175fe000000000016be7&sceneID=5
{
”ok”: true,
”result”:
{
”sceneID”: 5,
”dontCare”: false,
”localPrio”: false,
”specialMode”: false,
”flashMode”: false,
”ledconIndex”: 1,
”dimtimeIndex”: 1
}
}
4.6.7
Set Scene Mode
Sets the device configuration flags for a given sceneID. For details about the scene configuration see
the ds-basics reference document.
Synopsis
HTTP GET /json/device/setSceneMode
Parameter
72

Parameter
Description
Remarks
sceneID
Scene number which has been requested
Mandatory
dontCare
Don’t Care Flag
Optional
localPrio
Local Prio Flag
Optional
specialMode
Special Mode Flag
Optional
flashMode
Flashing Mode Flag
Optional
ledconIndex
Index of the LED configuration register
Optional
dimmtimeIndex Index of the transition configuration register Optional
Response
HTTP Status 200
ok true
Sample
GET /json/device/setSceneMode?dsid=3504175fe000000000016be7&sceneID=5&dimtimeIndex=2&dontCare=true
{
”ok”: true
}
4.6.8
Blink
Executes the ”blink” function on a device for identification purposes.
Synopsis
HTTP GET /json/device/blink
Parameter
None
Response
HTTP Status 200
ok true
Sample
GET /json/device/blink?dsid=3504175fe000000000017ef3
{
”ok”:true
}
4.6.9
Get Output Channel Value
Retrieve the value of one or more output channels of the device.
73

Synopsis
HTTP GET /json/device/getOutputChannelValue
Parameter
Parameter Description
Remarks
channels
Semicolon separated list of channel names Mandatory
Currently supported channels are listed below. For details please refer to the the dS-Basics document
in the section Output Channel Types.
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
Response
HTTP Status 200
result.channels
array of channels and their values
result.channels[x].channel output channel name
result.channels[x].value
output channel value
74

Sample
GET /json/device/getOutputChannelValue?dsid=3504175fe000000000016c4f&channels=brightness;saturation
{
”ok”:true,
result: {
channels: [
{ channel: ”brightness”, value: 50 },
{ channel: ”saturation”, value: 80 }
]
}
}
4.6.10
Set Output Channel Value
Set the value of one or more output channels of the device.
Synopsis
HTTP GET /json/device/setOutputChannelValue
Parameter
Parameter
Description
Remarks
channelvalues Semicolon separated list of channel names
and their values
Mandatory
applyNow
Immediately apply the new values to the
channe outputs
Optional,
1 (true) by
default
See getOutputChannelValue description 4.6.9 for a list of output channel names and their value ranges.
Response
HTTP Status 200
ok true
Sample
GET /json/device/setOutputChannelValue?dsid=3504175fe000000000016c4f&channelvalues=brightness=10;saturation=100&applyNow=1
{ ”ok”:true }
4.6.11
Get Output Channel Value v2
Retrieve the current output value of selected or all channels of the device.
Synopsis
HTTP GET /json/device/getOutputChannelValue2
75

Parameter
Parameter Description
Remarks
channels
Semicolon separated list of channel names Optional
If channels parameter is empty or omitted the call returns current values for all channels.
See getOutputChannelValue description 4.6.9 for a list of output channel names and their value ranges.
Response
HTTP Status 200
result.channels
array of channels and their values
result.channels[x].channel output channel name
result.channels[x].value
output channel value
Sample
GET /json/device/getOutputChannelValue2?dsuid=5a11caa06212578280d826428d15c3d700&channels=brightness;saturation;hue
{
”ok”:true,
result: {
channels: {
”brightness”: { ”value”: 50, ”automatic”: false },
”saturation”: { ”value”: 80 },
”hue”: { ”value”: 0 }
}
}
}
4.6.12
Set Output Channel Value v2
Set the value of one or all output channels of the device.
Synopsis
HTTP GET /json/device/setOutputChannelValue2
Parameter
Parameter Description
Remarks
channels
json object with channel names and their val-
ues
Mandatory
applyNow
Immediately apply the new values to the chan-
nels outputs
Optional,
1 (true) by
default
The channels parameter json object has the same structure like returned by getOutputChannelValue2.
See getOutputChannelValue description 4.6.9 for a list of output channel names and their value ranges.
76

WARNING: Values passed to setOutputChannelValue2 are not symmetric to values in getOutputChannel-
Value2 response. Client is responsible to scale the values passed to setOutputChannelValue2 to units
used at dsm-api as specified in ds-basics.pdf section Output Channel Types.
Response
HTTP Status 200
ok true
Sample
GET /json/device/setOutputChannelValue2?dsuid=5a11caa06212578280d826428d15c3d700&channels={”brightness”: {”value”: 10,
”automatic”: false}, ”saturation”: {”value”: 100}, ”hue”: {”value”: 235}}
{ ”ok”:true }
4.6.13
Get Output Channel Scene Value
Get scene value of one or more output channels.
Synopsis
HTTP GET /json/device/getOutputChannelSceneValue
Parameter
Parameter
Description
Remarks
channels
Semicolon separated list of channel names
Mandatory
sceneNumber Number of the scene for which the values
should be returned
Mandatory
See getOutputChannelValue description 4.6.9 for a list of output channel names and their value ranges.
Response
HTTP Status 200
result.sceneID
Scene number for which the values are returned
result.channels
array of channels and their values
result.channels[x].channel output channel name
result.channels[x].value
output channel value for the requested scene
Sample
GET /json/device/getOutputChannelSceneValue?dsid=3504175fe000000000016c4f&channels=brightness;saturation&sceneNumber=1
{
”ok”:true,
result: {
sceneID: 1,
77

channels: [
{ channel: ”brightness”, value: 40 },
{ channel: ”saturation”, value: 20 }
]
}
}
4.6.14
Set Output Channel Scene Value
Set scene value of one or more output channels.
Synopsis
HTTP GET /json/device/setOutputChannelSceneValue
Parameter
Parameter
Description
Remarks
channelvalues Semicolon separated list of channel names
and their values
Mandatory
sceneNumber Number of the scene for which the values
should be set
Mandatory
See getOutputChannelValue description 4.6.9 for a list of output channel names and their value ranges.
Response
HTTP Status 200
ok true
Sample
GET /json/device/setOutputChannelSceneValue?dsid=3504175fe000000000016c4f&channelvalues=brightness=10;saturation=100&
sceneNumber=1
{
”ok”:true
}
4.6.15
Get Output Channel Scene Value v2
Reads the device configuration for a given sceneID and output channels.
Synopsis
HTTP GET /json/device/getOutputChannelSceneValue2
Parameter
Parameter
Description
Remarks
sceneNumber Scene number for which the configuration is requested Mandatory
channels
Semicolon separated list of channel names
Mandatory
78

See getOutputChannelValue description 4.6.9 for a list of output channel names and their value ranges.
Response
HTTP Status 200
sceneID
Scene number which has been requested
channels
Object with one JSON object per available channel type
channels.[x].value
Numeric channel value
channels.[x].dontCare
Don’t Care Flag
channels.[x].automatic Automatic Operation Flag
Sample
GET /json/device/getOutputChannelSceneValue2?dsid=3504175fe000000000016be7&sceneNumber=5&
channels=airFlowIntensity;airLouverPosition
{
”ok”: true,
”result”:
{
”sceneID” : 5,
”channels” : {
”airFlowIntensity” : { ”value”: 0, ”dontCare”: false, ”automatic”: true},
”airLouverPosition” : { ”value”: 50, ”dontCare”: false, ”automatic”: false}
}
}
}
4.6.16
Set Output Channel Scene Value v2
Sets the device configuration flags for a given sceneID and output channels.
Synopsis
HTTP GET /json/device/setOutputChannelSceneValue2
Parameter
Parameter
Description
Remarks
sceneNumber Scene number which has been requested
Mandatory
channels
json object with channel names and their val-
ues
Mandatory
The channels parameter json object has the same structure like returned by getOutputChannelSceneValue2.
See getOutputChannelValue description 4.6.9 for a list of output channel names and their value ranges.
Response
HTTP Status 200
ok true
79

Sample
GET /json/device/setOutputChannelSceneValue2?dsid=3504175fe000000000016be7&sceneNumber=5&channels={ ”airFlowIntensity” :
{”dontCare”: true}, ”airLouverPosition” : {”value”: 100, ”automatic”: true}}
{
”ok”: true
}
4.6.17
Get Output Channel Don’t Care Flags
Get don’t care flags for one or more output channels.
Notice getOutputChannelDontCareFlag is DEPRECATED. Please use Get Output Channel Scene
Mode instead.
Synopsis
HTTP GET /json/device/getOutputChannelDontCareFlags
Parameter
Parameter
Description
Remarks
sceneNumber Scene number for which the flag will be set Mandatory
Response
HTTP Status 200
result.channels
array of channels and their values
result.channels[x].channel
output channel name
result.channels[x].dontCare output channel ”don’t care” flag value
Sample
GET /json/device/getOutputChannelDontCareFlags?dsid=3504175fe000000000016c4f&channels=brightness;saturation&dontCare=1&
sceneNumber=1
{
”ok”:true,
result: {
channels: [
{ channel: ”brightness”, dontCare: 0 },
{ channel: ”saturation”, dontCare: 1 }
]
}
}
4.6.18
Set Output Channel Don’t Care Flag
Set don’t care flag for one or more output channels.
Notice setOutputChannelDontCareFlag is DEPRECATED. Please use Set Output Channel Scene
Mode instead.
80

Synopsis
HTTP GET /json/device/setOutputChannelDontCareFlag
Parameter
Parameter
Description
Remarks
channels
Semicolon separated list of channel names Mandatory
dontCare
Don’t care flag value, boolean
Mandatory (0 or 1)
sceneNumber Scene number for which the flag will be set Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setOutputChannelDontCareFlag?dsid=3504175fe000000000016c4f&channels=brightness;saturation&dontCare=1&
sceneNumber=1
{
”ok”:true
}
4.7
Configuration
4.7.1
setButtonID
Sets the button ID of a device. For details about the push button configuration see the ds-basics refer-
ence document.
Synopsis
HTTP GET /json/device/setButtonID
Parameter
Parameter Description
Remarks
buttonID
Button number to set Mandatory
Response
HTTP Status 200
ok true
Sample
81

GET /json/device/setButtonID?dsid=3504175fe000000000016be7&buttonID=5
{
”ok”: true
}
4.7.2
setButtonInputMode
Sets the button input mode of a device. For details about the push button configuration see the ds-
basics reference document.
Synopsis
HTTP GET /json/device/setButtonInputMode
Parameter
Parameter Description
Remarks
modeID
Numerical value of the button mode to set Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setButtonInputMode?dsid=3504175fe000000000016be7&modeID=0
{
”ok”: true
}
4.7.3
setOutputMode
Sets the output mode of a device.
Synopsis
HTTP GET /json/device/setOutputMode
Parameter
Parameter Description
Remarks
modeID
Numerical value of the output mode to set Mandatory
Response
HTTP Status 200
ok true
82

Sample
GET /json/device/setOutputMode?dsid=3504175fe000000000016be7&modeID=0
{
”ok”: true
}
4.7.4
setJokerGroup
Sets the color group of a Joker device.
Synopsis
HTTP GET /json/device/setJokerGroup
Parameter
Parameter Description
Remarks
groupID
Group number to set Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setJokerGroup?dsid=3504175fe000000000016be7&groupID=2
{
”ok”: true
}
4.7.5
setButtonActiveGroup
Sets the user group of a push button device.
Synopsis
HTTP GET /json/device/setButtonActiveGroup
Parameter
Parameter Description
Remarks
groupID
Group number to set Mandatory, value range between 0 and 63,use 0xff to reset
Response
HTTP Status 200
ok true
83

Sample
GET /json/device/setButtonActiveGroup?dsid=3504175fe000000000016be7&groupID=20
{
”ok”: true
}
4.7.6
getTransitionTime
Reads the device transition time configuration for a given register set. For details about the transition
time configuration see the ds-basics reference document.
Synopsis
HTTP GET /json/device/getTransitionTime
Parameter
Parameter
Description
Remarks
dimtimeIndex Index of the transition configuration register Mandatory
Response
HTTP Status 200
dimmtimeIndex Index of the transition configuration register
up
Ramptime up in Milliseconds
down
Ramptime down in Milliseconds
Sample
GET /json/device/getTransitionTime?dsid=3504175fe000000000016be7&dimtimeIndex=2
{
”ok”: true,
”result”:
{
”dimtimeIndex”: 2,
”up”: 600,
”down”: 55
}
}
4.7.7
setTransitionTime
Sets the device transition time configuration for a given register set. For details about the transition
time configuration see the ds-basics reference document.
Synopsis
HTTP GET /json/device/setTransitionTime
84

Parameter
Parameter
Description
Remarks
dimtimeIndex Index of the transition configuration register Mandatory
up
Ramptime up in Milliseconds
Mandatory
down
Ramptime down in Milliseconds
Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setTransitionTime?dsid=3504175fe000000000016be7&dimtimeIndex=2&up=600&down=600
{
”ok”: true
}
4.7.8
setConfig
Write a configuration value of a config class parameter to the device.
Notice Writing configuration parameters directly to the device may lead to malfunctions including
complete failure of the whole device. Do not write parameters or values unless you are sure that
the device supports it.
Synopsis
HTTP GET /json/device/setConfig
Parameter
Parameter Description
Remarks
class
Configuration Class Mandatory
index
Parameter Index
Mandatory
value
Parameter Value
Mandatory
Response
HTTP Status 200
class the class parameter from the request
index the index parameter from the request
value parameter value
85

Sample
GET /json/device/setConfig?dsid=3504175fe000000000016be7&class=3&index=0&value=33
{
”ok”: true
}
4.7.9
getConfig
Reads a 8 bit parameter value of a config class from the device.
Notice Getting parameter values directly from the device takes a noticeable amount of time. This
request is subject of limitations in the systems certification rules.
Synopsis
HTTP GET /json/device/getConfig
Parameter
Parameter Description
Remarks
class
Configuration class Mandatory
index
Parameter index
Mandatory
Response
HTTP Status 200
class the class parameter from the request
index the index parameter from the request
value parameter value
Sample
GET /json/device/getConfig?dsid=3504175fe000000000016be7&class=1&index=2
{
”ok”: true,
”result”:
{
”class”: 1,
”index”: 2,
”value”: 231
}
}
4.7.10
getConfigWord
Reads a 16 bit parameter value of a config class from the device.
Notice Getting parameter values directly from the device takes a noticeable amount of time. This
request is subject of limitations in the systems certification rules.
86

Synopsis
HTTP GET /json/device/getConfigWord
Parameter
Parameter Description
Remarks
class
Configuration class
Mandatory
index
Parameter index, even Mandatory
Response
HTTP Status 200
class the class parameter from the request
index the index parameter from the request
value parameter value
Sample
GET /json/device/getConfigWord?dsid=3504175fe000000000016be7&class=3&index=2
{
”ok”: true,
”result”:
{
”class”: 3,
”index”: 2,
”value”: 65280
}
}
4.7.11
setCardinalDirection
Write the cardinal direction of the device.
Synopsis
HTTP GET /json/device/setCardinalDirection
Parameter
Parameter Description
Remarks
direction
the cardinal direction of this device. Allowed values:
none
north
north east
east
south east
south
south west
west
north west
87

Response
HTTP Status 200
Sample
GET /json/device/setCardinalDirection?dsid=3504175fe000000000016be7&direction=south\%20west
{
”ok”: true
}
4.7.12
getCardinalDirection
Read the configured cardinal direction of the device.
Synopsis
HTTP GET /json/device/getCardinalDirection
Response
HTTP Status 200
direction the cardinal direction of this device. Allowed values:
none
north
north east
east
south east
south
south west
west
north west
Sample
GET /json/device/getCardinalDirection?dsid=3504175fe000000000016be7
{
”ok”: true,
”result”:
{
”direction”: ”south￿west”
}
}
4.7.13
setWindProtectionClass
Write the wid protection class of the device.
Synopsis
HTTP GET /json/device/setWindProtectionClass
88

Parameter
Parameter Description
Remarks
class
the wind protection class of this device.
Response
HTTP Status 200
Sample
GET /json/device/setWindProtectionClass?dsid=3504175fe000000000016be7&class=4
{
”ok”: true
}
4.7.14
getWindProtectionClass
Read the the wid protection class of the device.
Synopsis
HTTP GET /json/device/getWindProtectionClass
Response
HTTP Status 200
class the wind protection class of this device.
Sample
GET /json/device/getWindProtectionClass?dsid=3504175fe000000000016be7
{
”ok”: true,
”result”:
{
”class”: 2
}
}
4.7.15
setFloor
Write floor number where the device is installed.
Synopsis
HTTP GET /json/device/setFloor
Parameter
Parameter Description
Remarks
floor
the floor number where the device is installed.
89

Response
HTTP Status 200
Sample
GET /json/device/setFloor?dsid=3504175fe000000000016be7&floor=14
{
”ok”: true
}
4.7.16
getFloor
Read floor number where the device is installed.
Synopsis
HTTP GET /json/device/getFloor
Response
HTTP Status 200
floor the floor number where the device is installed.
Sample
GET /json/device/getFloor?dsid=3504175fe000000000016be7
{
”ok”: true,
”result”:
{
”floor”: 14
}
}
4.7.17
getMaxMotionTime
Reads the maximum motion time configuration of a shade device.
Synopsis
HTTP GET /json/device/getMaxMotionTime
Parameter
None
Response
HTTP Status 200
result.supported boolean flag indicating if device supports this configuration
result.value
maximum motion time in seconds
90

Sample
GET /json/device/getMaxMotionTime?dsuid=3504175fe00000000000000000017bf600
{
”result”:
{
”supported”: false,
”value”: 0
},
”ok”: true
}
4.7.18
setMaxMotionTime
Configures the maximum motion time of a shade device.
Synopsis
HTTP GET /json/device/setMaxMotionTime
Parameter
Parameter Description
Remarks
seconds
Maximum motion time in seconds where (0 < seconds < 655) Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setMaxMotionTime?dsid=3504175fe000000000016be7&seconds=10
{
”ok”: true
}
4.7.19
getOutputAfterImpulse
Reads configuration of an UMR device output after an impulse.
Synopsis
HTTP GET /json/device/getOutputAfterImpulse
Parameter
None
Response
HTTP Status 200
result.output current output after impulse setting, can be ”on”, ”off” or ”retain”
91

Sample
HTTP GET /json/device/getOutputAfterImpulse?dsuid=302ed89f43f0000000000ec00009478a00
{
”result”:
{
”output”: ”retain”
},
”ok”: true
}
4.7.20
setOutputAfterImpulse
Configures UMR device output after an impulse.
Synopsis
HTTP GET /json/device/setOutputAfterImpulse
Parameter
Parameter Description
Remarks
output
output setting: ”on”, ”off” or ”retain” Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/getOutputAfterImpulse?dsuid=302ed89f43f0000000000ec00009478a00&output=off
{
”ok”: true
}
4.7.21
setVisibility
Configure TNY device visibility. Not allowed for ”main” device.
Synopsis
HTTP GET /json/device/setVisibility
Parameter
Parameter Description
Remarks
visibility
visibility setting: 0 or 1 Mandatory
92

Response
HTTP Status 200
result.action
one of ”add”, ”none”, ”remove”
result.devices array of devices to be processed
Sample
/json/device/setVisibility?dsuid=302ed89f43f00000000005400009f75d00&visibility=1
{
”result”:
{
”action”: ”none”
},
”ok”: true
}
4.7.22
setSupportedBasicScenes
Enables to provide a set of basic scenes supported by ventilation devices. Available scene numbers and
their descriptions are: 0 - ”Off”, 5 - ”Level1”, 17 - ”Level2”, 18 - ”Level3”, 19 - ”Level4”, 36 - ”Boost”.
Synopsis
HTTP GET /json/device/setSupportedBasicScenes
Parameter
Parameter Description
Remarks
value
Array of scene numbers Mandatory
Response
HTTP Status 200
ok true
Sample
/json/device/setSupportedBasicScenes?dsuid=302ed89f43f00000000005400009f75d00&value=[0,5,17,18,19,36]
{
”ok”: true
}
4.7.23
getSupportedBasicScenes
Reads out a set of basic scenes supported by ventilation device.
Synopsis
HTTP GET /json/device/getSupportedBasicScenes
93

Parameter
None
Response
HTTP Status 200
value Array of supported basic scenes
Sample
/json/device/getSupportedBasicScenes?dsuid=302ed89f43f00000000005400009f75d00
{
”result”:
{
”value”: [0,17,18]
},
”ok”: true
}
4.7.24
setIgnoreOperationLock
Sets ignore operation lock on a device
Synopsis
HTTP GET /json/device/setIgnoreOperationLock
Parameter
Parameter Description
Remarks
lockStatus lock status, boolean: 1 = lock ignored, 0 = lock observed Mandatory (0 or 1)
Response
HTTP Status 200
ok true
Sample
GET /json/device/setIgnoreOperationLock?dsuid=302ed89f43f0000000000ec00009478a00&lockStatus=1
{
”ok”: true
}
4.7.25
getIgnoreOperationLock
Gets ignore operation lock on a device
Synopsis
HTTP GET /json/device/getIgnoreOperationLock
94

Parameter
Response
HTTP Status 200
ignoreOperationLock information whether ignore operation lock is set
Sample
GET /json/device/getIgnoreOperationLock?dsuid=302ed89f43f0000000000ec00009478a00
{
”ok”: true
”result”:
{
”ignoreOperationlock”: false
}
}
4.8
Sensor
4.8.1
Get Sensor Value
Ready a sensor measurement from a device.
Synopsis
HTTP GET /json/device/getSensorValue
Parameter
Parameter
Description
Remarks
sensorIndex Numerical value, in the range from 0 to 14 Mandatory
Response
HTTP Status 200
sensorIndex the index parameter from the request
sensorValue the actual measurement read from the device
Sample
GET /json/device/getSensorValue?dsid=3504175fe000000000017ef3&sensorIndex=4
{
”ok”: true,
”result”: {
”sensorIndex”: 4,
”sensorValue”: 0
}
}
4.8.2
Get Sensor Type
Ready the sensor type description from a device. For details about sensor types see the ds-basics
reference document.
95

Synopsis
HTTP GET /json/device/getSensorType
Parameter
Parameter
Description
Remarks
sensorIndex Numerical value, in the range from 0 to 14 Mandatory
Response
HTTP Status 200
sensorIndex the index parameter from the request
sensorType
the sensor type read from the device
Sample
GET /json/device/getSensorType?dsid=3504175fe000000000017ef3&sensorIndex=4
{
”ok”: true,
”result”: {
”sensorIndex”: 4,
”sensorType”: 6
}
}
4.8.3
getSensorEventTableEntry
Reads the device event configuration for a given index. For details about the event table configuration
see the ds-basics reference document.
Synopsis
HTTP GET /json/device/getSensorEventTableEntry
Parameter
Parameter Description
Remarks
eventIndex Index of the event configuration entry Mandatory
Response
HTTP Status 200
eventIndex
Index of the event configuration register
eventName
User defined name of this event
sensorIndex Sensor index on which this entry operates
action
Action value
value
Threshold value
test
Comparison operator
hysteresis
Hysteresis value
validity
Enabled Flag
96

Sample
GET /json/device/getSensorEventTableEntry?dsid=3504175fe00000000001540c&eventIndex=0
{
”ok”: true,
”result”: {
”eventIndex”: 0,
”eventName”: ””,
”sensorIndex”: 2,
”test”: 2,
”action”: 0,
”value”: 35,
”hysteresis”: 0,
”validity”: 2
}
}
4.8.4
setSensorEventTableEntry
Sets the device event configuration for a given index. For details about the event table configuration
see the ds-basics reference document.
Synopsis
HTTP GET /json/device/setSensorEventTableEntry
Parameter
Parameter
Description
Remarks
eventIndex
Index of the event configuration register
Mandatory
eventName
User defined name of this event
Mandatory
sensorIndex Sensor index on which this entry operates Mandatory
action
Action value
Mandatory
value
Threshold value
Mandatory
test
Comparison operator
Mandatory
hysteresis
Hysteresis value
Mandatory
validity
Enabled Flag
Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setSensorEventTableEntry?dsid=3504175fe00000000001540c&eventIndex=0&eventName=”TV￿turned￿on”&
sensorIndex=2&test=2&action=0&value=50&hysteresis=25&validity=2
{
”ok”: true
}
4.9
Programming
4.9.1
Set Programming Mode
Enabled or disabled the programming mode on a device.
97

Synopsis
HTTP GET /json/device/setProgMode
Parameter
Parameter Description
Remarks
mode
mode value, either enabled or disabled Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setProgMode?dsid=3504175fe000000000017ef3&mode=disabled
{
”ok”: true
}
4.9.2
Add To Area
Modify the device scene table configuration and activate the area scene.
Synopsis
HTTP GET /json/device/addToArea
Parameter
Parameter Description
Remarks
areaScene either the area-on or area-off scenes Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/addToArea?dsid=3504175fe000000000017ef3&areaScene=7
{
”ok”: true
}
4.9.3
Remove From Area
Modify the device scene table configuration and deactivate the area scene.
Synopsis
HTTP GET /json/device/removeFromArea
98

Parameter
Parameter Description
Remarks
areaScene either the area-on or area-off scenes Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/removeFromArea?dsid=3504175fe000000000017ef3&areaScene=7
{
”ok”: true
}
4.9.4
Get Transmission Quality
Sends test commands to a device to evaluate the actual transmission quality.
Synopsis
HTTP GET /json/device/getTransmissionQuality
Parameter
None
Response
HTTP Status 200
upstream
a numerical value in the range of 0 to 62, 62 meaning best quality
downstream a numerical value in the range 0 to 6, 0 meaning best quality
Sample
GET /json/device/getTransmissionQuality?dsid=3504175fe000000000017ef3
{
”ok”: true,
”result”: {
”upstream”: 61,
”downstream”: 0
}
}
4.10
Heating and valve actuators
4.10.1
setHeatingGroup
Sets the standard color group of a heating actuator. Some actuators support operation with differ-
ent connected hardware equipment, therefore the terminal blocks support operation in different zone
groups, for example in heating, cooling or ventilation.
99

Synopsis
HTTP GET /json/device/setHeatingGroup
Parameter
Parameter Description
Remarks
groupID
New group Id Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setHeatingGroup?dsuid=3504175fe00000000000000000016be700&groupID=48
{
”ok”: true
}
4.10.2
getValvePwmState
Reads the device status of a valve PWM actuator.
Synopsis
HTTP GET /json/device/getValvePwmState
Parameter
None
Response
HTTP Status 200
pwmValue
Current PWM control value in percent
pwmPriorityMode Current operating state value
Sample
GET /json/device/getValvePwmState?dsuid=3504175fe00000000000000000016be700
{
”ok”: true,
”result”:
{
”pwmValue”: 60,
”pwmPriorityMode”: 0
}
}
4.10.3
getValvePwmMode
Reads the device configuration of a valve PWM actuator.
100

Synopsis
HTTP GET /json/device/getValvePwmMode
Parameter
None
Response
HTTP Status 200
pwmPeriod Length of PWM period in seconds
pwmMinX
Minimum set point or threshold
pwmMaxX
Maximum set point or threshold
pwmMinY
Minimum output value at min set point
pwmMaxY
Maximum output value at max set point
pwmOffset
Set point offset
Sample
GET /json/device/getValvePwmMode?dsuid=3504175fe00000000000000000016be700
{
”ok”: true,
”result”:
{
”pwmPeriod”: 900,
”pwmMinX”: 0,
”pwmMaxX”: 10,
”pwmMinY”: 12,
”pwmMaxY”: 28,
”pwmOffset”: −20
}
}
4.10.4
setValvePwmMode
Sets the device configuration of a valve PWM actuator.
Synopsis
HTTP GET /json/device/setValvePwmMode
Parameter
Parameter
Description
Remarks
pwmPeriod Length of PWM period in seconds, 0 .. 64k
Optional
pwmMinX
Minimum set point or threshold, 0 .. 255
Optional
pwmMaxX
Maximum set point or threshold, 0 .. 255
Optional
pwmMinY
Minimum output value at min set point, 0 .. 255
Optional
pwmMaxY
Maximum output value at max set point, 0 .. 255 Optional
pwmOffset
Set point offset, -128 .. 127
Optional
101

Response
HTTP Status 200
ok true
Sample
GET /json/device/setValvePwmMode?dsuid=3504175fe00000000000000000016be700&pwmMaxX=80&pwmOffset=−20
{
”ok”: true
}
4.10.5
getValveControlMode
Reads the device configuration of a valve PWM actuator.
Synopsis
HTTP GET /json/device/getValveControlMode
Parameter
None
Response
HTTP Status 200
ctrlNONC
Configure normally closed (false) or normally open (true) output behavior
ctrlClipMaxHigher PWM value over maximum control value to 100 percent
ctrlClipMinLower
PWM lower than maximum control value to 0 percent
ctrlClipMinZero
Control value of zero maps to 0 percent PWM
Sample
GET /json/device/getValveControlMode?dsuid=3504175fe00000000000000000016be700
{
”ok”: true,
”result”:
{
”ctrlNONC”: true,
”ctrlClipMaxHigher”: false,
”ctrlClipMinLower”: false,
”ctrlClipMinZero”: false
}
}
4.10.6
setValveControlMode
Sets the device configuration of a valve PWM actuator.
Synopsis
HTTP GET /json/device/setValveControlMode
102

Parameter
Parameter
Description
Remarks
ctrlClipMinZero
Control value of zero maps to 0 percent PWM
Optional
ctrlClipMinLower
PWM lower than minimum forces control value
to 0 percent
Optional
ctrlClipMaxHigher PWM value over maximum forces control value
to 100 percent
Optional
ctrlNONC
Configure normally open or normally closed
output behavior
Optional
Response
HTTP Status 200
ok true
Sample
GET /json/device/setValveControlMode?dsuid=3504175fe00000000000000000016be700&ctrlNONC=false
{
”ok”: true
}
4.10.7
getValveTimerMode
Reads the timer device configuration settings of a valve PWM actuator.
Synopsis
HTTP GET /json/device/getValveTimerMode
Parameter
None
Response
HTTP Status 200
valveProtectionTimer Duration of the valve protection period in seconds
emergencyValue
Fixed output value in percent in emergency mode
emergencyTimer
Duration in seconds until emergency mode is activated
Sample
GET /json/device/getValveTimerMode?dsuid=3504175fe00000000000000000016be700
{
”ok”: true,
”result”:
{
”valveProtectionTimer”: 0,
”emergencyValue”: 75,
”emergencyTimer”: 7200
103

}
}
4.10.8
setValveTimerMode
Sets the timer device configuration of a valve PWM actuator.
Synopsis
HTTP GET /json/device/setValveTimerMode
Parameter
Parameter
Description
Remarks
valveProtectionTimer Duration of the valve protection period in sec-
onds
Optional
emergencyValue
Fixed output value in percent in emergency
mode
Optional
emergencyTimer
Duration in seconds until emergency mode is
activated
Optional
Response
HTTP Status 200
ok true
Sample
GET /json/device/setValveTimerMode?dsuid=3504175fe00000000000000000016be700&valveProtectionTimer=600
{
”ok”: true
}
4.11
Single Device Info
The Single Device Info section refers to the device description data that is available only for selected
devices. Please read the Device Description section of the system interfaces documentation.
If the device is not a Single Device with device description data the following error message will be
returned:
{
”ok”: false,
”message”: ”Device￿does￿not￿support￿action￿configuration”
}
4.11.1
Get Info Static
Returns the static device description data of a device. This data is available in a database on the
digitalSTROM-Server and is not fetched from the device itself.
Synopsis
HTTP GET /json/device/getInfoStatic
104

Parameter
Parameter Description
Remarks
lang
Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory
Response
HTTP Status 200
spec
common device description object
stateDescriptions
list of state description objects
eventDescriptions
list of event description objects
propertyDescriptions list of property description objects
sensorDescriptions
list of sensor description objects
actionDescriptions
list of action description objects
standardActions
list of standard action description objects
spec
”spec”: {
”descriptionId”: {
”title”: ”Translated￿title￿for￿key￿descriptionId”,
”value”: ”Value￿of￿the￿key￿descriptionId”,
”tags”: ”string￿value,￿semi−colon￿separated￿list￿of￿attributes”
}
}
Common descriptionId’s are:
”name”
User given name of the dSDevice
”dsDeviceGTIN”
GTIN of the dSDevice
”model”
Product Name
”modelVersion”
Product/Model Revision
”vendorName”
Vendor/Maker
”vendorId”
Vendor ID, numerical number
”hardwareGuid”
Instance ID of the hardware, e.g. S/N, MAC Address, SGTIN
”hardwareModelGuid” Model ID of the hardware, e.g. GTIN of the Native Device
”class”
dS Class/Profile Name, e.g. ”Water Kettle”, ”Dishwasher”
”classId”
dS Class/Profile ID
”classVersion”
Revision Number of the supported class/profile
stateDescriptions
”stateTechnicalName”: {
”title”: ”Translated￿title￿for￿this￿state￿object”,
”options”: { list of ”OptionId”: ”OptionValue” pairs }
”tags”: ”optional￿string￿value,￿semi−colon￿separated￿list￿of￿attributes”
}
Example:
105

”operation”: {
”title”: ”Betriebszustand”,
”options”: {
”idle”: ”Bereitschaft”,
”active”: ”Aktiv”,
”error”: ”Fehler”
},
”tags”: null
}
eventDescriptions
”eventTechnicalName”: {
”title”: ”Translated￿title￿for￿this￿event￿object”,
”accessLevel”: optional enum of
”normal” (default) // Shown everywhere
”advanced” // Shown within the UI if the ‘show more ’events option was selected
”service” // Not shown within the ’UIs at all but might be used by bots and other services.
}
propertyDescriptions
”parameterTechnicalName”: {
”title:” ”Translated￿name￿of￿this￿parameter”,
”type”: ”data￿type￿of￿the￿parameter￿value:￿numeric,￿enumeration,￿string”,
”tags”: ”string￿value,￿semi−colon￿separated￿list￿of￿attributes”
}
Additional optional fields for type numeric:
min
minimum value
max
maximum value
resolution minimum step size
siunit
unit string, http://www.ebyte.it/library/educards/siunits/TablesOfSiUnitsAndPrefixes.html, e.g. ”sec
default
a default value of the property
Additional optional fields for type enumeration:
options json object with a list of ”OptionId”: ”OptionValue” pairs
default a default value of the property
Additional optional fields for type text:
max
maximum length of the string value
default a default value of the property
Following fields are defined for the tags attribute:
readonly parameter value can only be read and not written
invisible
parameter shall not be shown and must be hidden in the UI
overview state or property shall be shown on the overview tab in the UI, order/position can be given with ”:num
settings
state or property shall be shown on the settings tab in the UI, order/position can be given with ”:num
106

The ”type” field string might have a postfix that indicates the characteristic of the value. This can be
used for rendering the data field in user interfaces.
Following postfix descriptions are defined for the type attribute:
numeric.timeOfDay hh:mm or am/pm depending on the region setting, does not include time zone
numeric.duration
hh:mm:ss or hh:mm, depending on unit and resolution
numeric.boolean
true/false, displayed as checkbox
Example:
”waterhardness”: {
”title”: ”Wasserhärte”,
”type”: ”numeric”,
”min”: 0,
”max”: 6,
”resolution”: 0.1,
”default”: 2.1,
”tags”: ”readonly”
}
sensorDescriptions
”sensorTechnicalName”: {
”title:” ”Translated￿name￿of￿this￿measurement”,
”type”: ”data￿type￿of￿the￿measurement￿value:￿numeric,￿enumeration,￿string”,
”tags”: ”string￿value,￿semi−colon￿separated￿list￿of￿attributes”
}
The sensor object is represented by the same extended fields then property objects, depending on the
”type” field.
Additional mandatory fields for sensor objects are:
dsType
device sensor type id number as defined by dS
dsIndex device index of the source, necessary to address in queries
actionDescriptions
”actionDescriptions”: {
”actionId1”: {
”title”: ”Translated￿label￿for￿actionId1”,
”params”: list of {propertyDescriptions}
},
....,
”actionIdN”: {
”title”: ”Translated￿label￿for￿actionIdN”,
”params”: list of {propertyDescriptions}
}
},
standardActions
”standardActions”: {
”std.Action1” : {
”action”: ”reference￿to￿the￿base￿action￿description”,
”title”: ”Translated￿name￿for￿std.Action1”,
”params”: { list of ”ParameterName”: ParameterValue, ...}
},
107

...,
”std.ActionN” : {
”action”: ”reference￿to￿the￿base￿action￿description”,
”title”: ”Translated￿name￿for￿std.ActionN”,
”params”: { list of ”ParameterName”: ParameterValue, ...}
}
}
Sample
GET /json/device/getInfoStatic?dsuid=687ba4e345e75bd58093bf119f8a6c6700&lang=de_DE
{
”result”: {
”spec”: {
”dsDeviceGTIN”: {
”title”: ”dS￿Device￿GTIN”,
”tags”: ”settings:5”,
”value”: ”7640156791945”
},
”hardwareGuid”: {
”title”: ”Artikel￿Kennzeichnung”,
”tags”: ”settings:4”,
”value”: ”MAC￿5C:CF:7F:11:F8:B8”
},
”hardwareModelGuid”: {
”title”: ”Produkt￿Kennzeichnung”,
”tags”: ”invisible”,
”value”: ”smartermodel:ikettle2”
},
”model”: {
”title”: ”Modell”,
”tags”: ”overview:2;settings:2”,
”value”: ”iKettle￿2”
},
”modelVersion”: {
”title”: ”Modellvariante”,
”tags”: ”invisible”,
”value”: ”19”
},
”name”: {
”title”: ”Name”,
”tags”: ”overview:1;settings:1”,
”value”: ”Wasserkocher”
},
”notes”: {
”title”: ”Bemerkungen”,
”tags”: ”overview:4”,
”value”: ”Bitte￿prüfen￿Sie￿mit￿der￿’Smarter’￿Smartphone￿App,￿ob￿die￿iKettle￿Firmware￿auf￿dem￿aktuellsten￿Stand￿ist!”
},
”vendorId”: {
”title”: ”Hersteller￿Kennung”,
”tags”: ”invisible”,
”value”: ”vendorname:Smarter￿Applications￿Ltd.”
},
”vendorName”: {
”title”: ”Hersteller”,
”tags”: ”overview:3;settings:3”,
”value”: ”Smarter￿Applications￿Ltd.”
},
”class”: {
”title”: ”Geräteklasse”,
”tags”: ”invisible”,
”value”: ””
},
”classVersion”: {
”title”: ”Geräteklassen￿Version”,
”tags”: ”invisible”,
”value”: ””
}
},
”stateDescriptions”: {
”operation”: {
”title”: ”Betriebsmodus”,
”tags”: ”overview”,
”options”: {
108

”cooldown”: ”Abkühlen”,
”heating”: ”Aufheizen”,
”keepwarm”: ”Warmhalten”,
”ready”: ”Bereit”,
”removed”: ”Abgehoben”
}
}
},
”eventDescriptions”: {
”KettleAttached”: {
”title”: ”Kocher￿aufgesetzt”
},
”BoilingStarted”: {
”title”: ”Aufheizen￿gestartet”
},
”KeepWarm”: {
”title”: ”Warmhalten￿gestartet”
},
”BabycoolingStarted”: {
”title”: ”Aufheizen￿beendet,￿￿auf￿Zieltemperatur￿abkühlen”
},
”BoilingFinished”: {
”title”: ”Aufheizen￿beendet”
},
”KettleReleased”: {
”title”: ”Kocher￿abgehoben”
},
”BabycoolingFinished”: {
”title”: ”Abkühlen￿beendet,￿Zieltemperatur￿erreichet”
},
”KeepWarmFinished”: {
”title”: ”Warmhalten￿beendet”
},
”BoilingAborted”: {
”title”: ”Aufheizen￿abgebrochen,￿Taste￿betätigt”
},
”BabycoolingAborted”: {
”title”: ”Abkühlen￿abgebrochen,￿Taste￿betätigt”
},
”KeepwarmAborted”: {
”title”: ”Warmhalten￿abgebrochen,￿Taste￿betätigt”
},
”KeepWarmAfterBoiling”: {
”title”: ”Aufheizen￿beendet,￿warmhalten”
},
”KeepWarmAfterBabycooling”: {
”title”: ”Abkühlen￿beendet,￿warmhalten”
},
”BoilingAbortedAndKettleReleased”: {
”title”: ”Aufheizen￿abgebrochen,￿Kocher￿abgehoben”
},
”BabyCoolingAbortedAndKettleReleased”: {
”title”: ”Abkühlen￿abgebrochen,￿Kocher￿abgehoben”
},
”KeepWarmAbortedAndKettleReleased”: {
”title”: ”Warmhalten￿abgebrochen,￿Kocher￿abgehoben”
}
},
”propertyDescriptions”: {
”currentTemperature”: {
”title”: ”Wassertemperatur”,
”tags”: ”readonly;overview”,
”type”: ”numeric”,
”min”: ”0”,
”max”: ”100”,
”resolution”: ”1”,
”siunit”: ”celsius”,
”default”: ”0”
},
”waterLevel”: {
”title”: ”Füllstand”,
”tags”: ”readonly;overview”,
”type”: ”numeric”,
”min”: ”0”,
”max”: ”2.0”,
”resolution”: ”0.2”,
”siunit”: ”liter”,
”default”: ”0”
109

},
”defaulttemperature”: {
”title”: ”Temperatur￿Aufheizen”,
”tags”: ”settings”,
”type”: ”numeric”,
”min”: ”0”,
”max”: ”100”,
”resolution”: ”1”,
”siunit”: ”celsius”,
”default”: ”100”
},
”defaultcooldowntemperature”: {
”title”: ”Temperatur￿Abkochen￿und￿Abkühlen”,
”tags”: ”invisible”,
”type”: ”numeric”,
”min”: ”0”,
”max”: ”100”,
”resolution”: ”1”,
”siunit”: ”celsius”,
”default”: ”80”
},
”defaultkeepwarmtime”: {
”title”: ”Warmhaltezeit”,
”tags”: ”settings”,
”type”: ”numeric”,
”min”: ”0”,
”max”: ”30”,
”resolution”: ”1”,
”siunit”: ”min”,
”default”: ”15”
}
},
”sensorDescriptions”: {
”waterQuantity”: {
”title”: ”Wassermenge”,
”tags”: ”readonly;overview”,
”type”: ”numeric”,
”min”: ”0”,
”max”: ”8”,
”resolution”: ”0.1”,
”siunit”: ”liter”,
”dsType”: 68,
”dsIndex”: 3
}
},
”actionDescriptions”: {
”boilandcooldown”: {
”title”: ”Abkochen￿und￿abkühlen”,
”params”: {
”keepwarmtime”: {
”title”: ”Warmhaltedauer”,
”tags”: ””,
”type”: ”numeric”,
”min”: ”0”,
”max”: ”30”,
”resolution”: ”1”,
”siunit”: ”min”,
”default”: ”30”
},
”temperature”: {
”title”: ”Zieltemperatur”,
”tags”: ””,
”type”: ”numeric”,
”min”: ”20”,
”max”: ”100”,
”resolution”: ”1”,
”siunit”: ”celsius”,
”default”: ”50”
}
}
},
”heat”: {
”title”: ”Aufheizen”,
”params”: {
”keepwarmtime”: {
”title”: ”Warmhaltedauer”,
”tags”: ””,
”type”: ”numeric”,
110

”min”: ”0”,
”max”: ”30”,
”resolution”: ”1”,
”siunit”: ”min”,
”default”: ”30”
},
”temperature”: {
”title”: ”Zieltemperatur”,
”tags”: ””,
”type”: ”numeric”,
”min”: ”20”,
”max”: ”100”,
”resolution”: ”1”,
”siunit”: ”celsius”,
”default”: ”100”
}
}
},
”stop”: {
”title”: ”Abschalten”,
”params”: {}
}
},
”standardActions”: {
”std.boilandcooldown”: {
”title”: ”Abkochen￿und￿abkühlen”,
”action”: ”boilandcooldown”,
”params”: {
”temperature”: ”40”
}
},
”std.heat”: {
”title”: ”Aufheizen”,
”action”: ”heat”,
”params”: {}
},
”std.stop”: {
”title”: ”Abschalten”,
”action”: ”stop”,
”params”: {}
}
}
},
”ok”: true
}
4.11.2
Get Info Operational
Returns the current value for states and properties.
Synopsis
HTTP GET /json/device/getInfoOperational
Parameter
Parameter Description
Remarks
lang
Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory
Response
HTTP Status 200
states
list of state value objects
properties list of property value objects
111

States and Property names are corresponding to the response of the static descriptions. The static
response contains translations and other meta information.
states
”states”: {
”stateTechnialName1”: {
”value”: ”stateOptionValue”
},
...,
”stateTechnicalNameN”: {
”value”: ”stateOptionValue”
}
},
properties
”properties”: {
”propertyTechnialName1”: {
”value”: number
},
...,
”propertyTechnicalNameN”: {
”value”: number
}
},
measurements
”sensors”: {
”sensorTechnialName1”: {
”value”: number
},
...,
”sensorTechnicalNameN”: {
”value”: number
}
},
Sample
GET /json/device/getInfoOperational?dsuid=687ba4e345e75bd58093bf119f8a6c6700&lang=de_DE
{
”result”: {
”states”: {
”operation”: {
”age”: 1.756285,
”changed”: 45452.848575,
”value”: ”ready”
}
},
”properties”: {
”currentTemperature”: 25,
”defaultkeepwarmtime”: 30,
”defaulttemperature”: 100,
”waterLevel”: 1.8620689655172413
},
”sensors”: {
”waterQuantity”: 8.6
}
},
”ok”: true
}
112

4.11.3
Get Info Custom
Returns the custom actions defined by the user or define. The custom actions are configurable and are
available in addition to the standard actions.
Synopsis
HTTP GET /json/device/getInfoCustom
Parameter
Parameter Description
Remarks
lang
Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory
Response
HTTP Status 200
customActions list of action description objects
The custom action name is given by the user. Each custom action is based on a defined and known
actionDescription of the device.
customActions
”customActions”: {
”custom.753151”: {
”action”: ”reference￿to￿the￿base￿action￿description”,
”title”: ”User￿given￿name￿for￿custom.753151”,
”params”: { list of ”ParameterName”: ParameterValue, ...}
},
....,
”custom.143937”: {
”action”: ”reference￿to￿the￿base￿action￿description”,
”title”: ”User￿given￿name￿for￿custom.143937”,
”params”: { list of ”ParameterName”: ParameterValue, ...}
}
}
Sample
GET /json/device/getInfoCustom?dsuid=687ba4e345e75bd58093bf119f8a6c6700&lang=de_DE
{
”result”: {
”customActions”: {
”custom.582620628227b”: {
”action”: ”std.boilandcooldown”,
”params”: {
”keepwarmtime”: 30,
”temperature”: 70
},
”title”: ”Früchtetee”
},
”custom.5826208698525”: {
”action”: ”std.heat”,
”params”: {
”keepwarmtime”: 0,
”temperature”: 42
},
”title”: ”Lauwarmes￿Wasser”
},
”custom.582620a92e329”: {
”action”: ”std.heat”,
113

”params”: {
”keepwarmtime”: 15,
”temperature”: 66
},
”title”: ”Spülwasser￿aufwärmen”
},
”custom.58404582ef972”: {
”action”: ”std.boilandcooldown”,
”params”: {
”keepwarmtime”: 0,
”temperature”: 100
},
”title”: ”Wasser￿abkochen”
}
}
},
”ok”: true
}
4.11.4
Get Info
getInfo is a method that combines static, operational and custom information in one call. With the given
filter parameter the caller can select which response fields he likes to have in the response.
Synopsis
HTTP GET /json/device/getInfo
Parameter
Parameter Description
Remarks
lang
Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Mandatory
filter
string with a comma separated list of response objects
Optional
The filter parameter accepts the following options: spec, standardActions, customActions, stateDesc,
propertyDesc, sensorDesc, actionDesc, eventDesc, operational.
If filter parameter is omitted or empty the full set of response objects is returned.
Response
HTTP Status 200
The response is a combination of the getInfoStatic, getInfoCustom and getInfoOperational response.
Please refer to the descriptions above.
4.11.5
Set Property
This method allows to change property values that are part of a getInfo Property Description.
Synopsis
HTTP GET /json/device/setProperty
Parameter
114

Parameter Description
Remarks
dsuid
dSUID of the device
Mandatory
id
property identifier
Mandatory
value
new value of the property Mandatory
The id parameter corresponds to the property technical name from getInfo response.
Response
HTTP Status 200
ok true
Sample
GET /json/device/setProperty?dsuid=687ba4e345e75bd58093bf119f8a6c6700&id=defaultkeepwarmtime&value=90.5
{
”ok”: true
}
4.11.6
Set Custom Action
This method allows to create or replace custom actions of a device.
Synopsis
HTTP GET /json/device/setCustomAction
Parameter
Parameter Description
Remarks
id
unique custom action identifier, must have prefix ”custom.”
Mandatory
title
user given name or title of this action
Mandatory
action
reference to basic action description identifier
Mandatory
params
json object with a list of property values: ”PropertyName”: ”PropertyValue” , ...
Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/setCustomAction?dsuid=687ba4e345e75bd58093bf119f8a6c6700&id=custom.123456&title=Lauwarmes Wasser&
action=std.heat&params={”temperature”:40}
{
”ok”: true
}
4.11.7
Call Action
Excutes the action id on a device.
115

Synopsis
HTTP GET /json/device/callAction
Parameter
Parameter Description
Remarks
id
standard or custom action identifier Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/device/callAction?dsuid=df6aa5bba4db5540c0fe55e3eb088be900&id=std.stop
{
”ok”:true
}
4.11.8
Get Apartment Scenes
Retrieves the list of device values for all supported apartment scenes (sceneID 64 and above).
Synopsis
HTTP GET /json/device/getSceneValue
Parameter
None
Response
HTTP Status 200
result.scenes a list of json objects per sceneID
Sample
GET /json/device/getApartmentScenes?dsuid=df6aa5bba4db5540c0fe55e3eb088be900
”result”: {
”scenes”: {
”64”: {
”channels”: null,
”command”: ”std.stop”,
”dontCare”: false,
”effect”: 1,
”ignoreLocalPriority”: true
},
....,
”92”: {
”channels”: null,
116

”command”: ”std.stop”,
”dontCare”: false,
”effect”: 1,
”ignoreLocalPriority”: true
}
}
},
”ok”: true
}
4.12
Test presence
4.12.1
testPresence
Tests the device presence.
Synopsis
HTTP GET /json/device/testPresence
Parameter
None
Response
HTTP Status 200
result.active bool - is the device active / inactive?
Sample
GET /json/device/getName?dsid=3504175fe000000000017ef3
{
”ok”:true,
”result” :
{
”active” : true
}
}
4.13
Button Usage
Most devices have a button input, but not all of them are connected to push-buttons. A user interface
may only display button inputs which were used before, ignoring all the terminal blocks where no but-
tons are connected to. A button is ”used” if it got pushed before, ”auto_unused” if it got not pushed yet,
or ”manual_unused” if a user set it into this state.
4.13.1
getButtonUsage
Returns information about button input usage of the device, only supported on devices that have button
inputs.
Synopsis
HTTP GET /json/device/getButtonUsage
Parameter
None
117

Response
HTTP Status 200
result.buttonUsage button usage setting (used, auto_unused, manual_unused, unsupported)
Sample
GET /json/device/getButtonUsage?dsuid=3504175fe0000000000000016c4f00
{
”ok”:true,
”result” :
{
”buttonUsage” : ”used”
}
}
4.13.2
setButtonUsage
Sets the button usage state, only supports devices that have buttons.
Synopsis
HTTP GET /json/device/setButtonUsage
Parameter Description
Remarks
usage
usage identifier (used, auto_unused, manual_unused) Mandatory
Parameter
Response
HTTP Status 200
ok true
Sample
GET /json/device/setButtonUsage?dsuid=3504175fe0000000000000016c4f00&usage=manual_unused
{
”ok”:true
}
118

