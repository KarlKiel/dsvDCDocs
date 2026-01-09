2 Apartment
===========

2
Apartment
2.1
Name
2.1.1
getName
Returns the user defined name of the installation.
Synopsis
HTTP GET /json/apartment/getName
Parameter
None
Response
HTTP Status 200
name identifier string for the installation
Sample
GET /json/apartment/getName
{
”ok”:true,
”result” :
{
”name” : ”digitalStrom￿Installation￿Hans￿Mustermann”
}
}
2.1.2
setName
Sets the installation name.
Synopsis
HTTP GET /json/apartment/setName
Parameter Description
Remarks
newName
identifier string for the installation Mandatory
Parameter
Response
HTTP Status 200
ok true
Sample
GET /json/apartment/setName?newName=”My￿dSS”
{
”ok”:true
}
10

2.2
Scene
2.2.1
callScene
Excutes the scene sceneNumber on a group of devices.
Synopsis
HTTP GET /json/apartment/callScene
Parameter
Parameter
Description
Remarks
sceneNumber Numerical value
Mandatory
groupID
Number of the target group
Optional
groupName
Name of the target group
Optional
force
Boolean value, if set a forced scene call is issued Optional
If the group parameters are omitted the command is sent as broadcast to all zones and all devices.
Response
HTTP Status 200
ok true
Sample
GET /json/apartment/callScene?sceneNumber=65
{
”ok”:true
}
2.2.2
saveScene
Tells devices to store their current output values as a default for the scene sceneNumber.
Synopsis
HTTP GET /json/apartment/saveScene
Parameter
Parameter
Description
Remarks
sceneNumber Numerical value
Mandatory
groupID
Number of the target group Optional
groupName
Name of the target group
Optional
If the group parameters are omitted the command is sent as broadcast to all zones and all devices.
11

Response
HTTP Status 200
ok true
Sample
GET /json/apartment/saveScene?sceneNumber=65
{
”ok”:true
}
2.2.3
undoScene
Tells devices to restore their output values to the previous state if the current scene matches the sce-
neNumber.
Synopsis
HTTP GET /json/apartment/undoScene
Parameter
Parameter
Description
Remarks
sceneNumber Numerical value
Mandatory
groupID
Number of the target group Optional
groupName
Name of the target group
Optional
If the group parameters are omitted the command is sent as broadcast to all zones and all devices.
Response
HTTP Status 200
ok true
Sample
GET /json/apartment/undoScene?sceneNumber=65
{
”ok”:true
}
2.2.4
getLockedScenes
Retrieves scene numbers of scenes that are currently locked because of an update of device scene
tables.
Synopsis
HTTP GET /json/apartment/getLockedScenes
12

Parameter
None
Response
HTTP Status 200
result.lockedScenes[] array of scene numbers that are currently locked
Sample
GET /json/apartment/getLockedScenes
{
”ok” : true,
”result” :
{
”lockedScenes” : []
}
}
2.3
Value
2.3.1
Set Device Output Value
Set the output value of a group of devices to a given value.
Notice Setting output values directly bypasses the group state machine and is unrecommended.
Synopsis
HTTP GET /json/apartment/setValue
Parameter
Parameter
Description
Remarks
value
Numerical value
Mandatory
groupID
Number of the target group Optional
groupName Name of the target group
Optional
If the group parameters are omitted the command is sent as broadcast to all devices.
Notice Setting output values without a group identification is strongly unrecommended.
Response
HTTP Status 200
ok
true
data array of devices that have binary inputs
Sample
13

GET /json/apartment/setValue?value=0&groupID=2
{
”ok”:true,
}
2.3.2
Get Binary Input Information
Retrieve the information about binary inputs of all devices.
Synopsis
HTTP GET /json/apartment/getDeviceBinaryInputs
Parameter
None
Response
HTTP Status 200
ok
true
devices array of devices that have binary inputs
Sample
GET /json/apartment/getDeviceBinaryInputs
{
”result”:
{
”devices”:
[
{
”dsuid”: ”3504175fe0000000000000000000d91100”,
”binaryInputs”:
[
{
”targetGroupType”: 0,
”targetGroup”: 8,
”inputType”: 11,
”inputId”: 15,
”state”: 1
}
]
},
{
”dsuid”: ”3504175fe0000000000000000000d91000”,
”binaryInputs”:
[
{
”targetGroupType”: 0,
”targetGroup”: 16,
”inputType”: 8,
”inputId”: 15,
”state”: 1
}
]
}
]
},
”ok”: true
}
2.4
Groups
14

2.4.1
getReachableGroups
Returns a list of groups for which are actuators actually present in the installation.
Synopsis
HTTP GET /json/apartment/getReachableGroups
Parameter
None
Response
HTTP Status 200
result.zones
array of zones in the installation
result.zones[].groups array of groups in a zone
Sample
GET /json/apartment/getReachableGroups
{
”ok”: true,
”result”: {
”zones”: [
{
”zoneID”: 0,
”name”: ””,
”groups”: [
64,
69
]
},
{
”zoneID”: 1223,
”name”: ”Wohnen”,
”groups”: [
1,
2,
7
]
},
{
”zoneID”: 1241,
”name”: ”Schlafen”,
”groups”: [
1,
5,
7
]
},
{
”zoneID”: 1237,
”name”: ”Essen”,
”groups”: [
1,
6
]
}
]
}
}
2.5
Structure
15

2.5.1
getStructure
Returns an object containing the structure of the apartment. This includes detailed information about
all zones, groups and devices.
Synopsis
HTTP GET /json/apartment/getStructure
Parameter
None
Response
HTTP Status 200
result.apartment.zones
array of zone information
result.apartment.zones[].devices
array of device information in each
zone
result.apartment.zones[].devices[].
groups
group membership of each device
result.apartment.zones[].groups
array of group information in each
zone
result.apartment.zones[].groups[].
devices
array of devices per group in a zone
result.apartment.zones[].groups[].
devices[x].modelFeatures
object of device specific model fea-
tures. These model features have
the same format as returned from
the getModelFeatures call (see 2.9)
and override the features given
there for this specific device. This
node is optional.
result.apartment.clusters
array of cluster information
result.apartment.clusters[].devices[].
clusters
cluster membership of each device
result.apartment.floors
array of floor information
Sample
GET /json/apartment/getStructure
{
”ok”: true,
”result”: {
”apartment”: {
”zones”: [
{
”id”: 0,
”name”: ””,
”isPresent”: false,
”floor”: 0,
”devices”: [
{
”id”: ”3504175fe0000000000182f6”,
”name”: ”Regalleuchte”,
”functionID”: 4152,
”productRevision”: 49955,
”productID”: 6344,
”hwInfo”: ”GE−SDS200”,
16

”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 97,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 22,
”buttonID”: 0,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
],
”modelFeatures”: {
”dontcare”: true,
”blink”: false,
”ledauto”: true
}
},
{
”id”: ”3504175fe00000000000439c”,
”name”: ”Stehlampe”,
”functionID”: 4152,
”productRevision”: 789,
”productID”: 200,
”hwInfo”: ”GE−KM200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 153,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 22,
”buttonID”: 0,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
]
},
{
”id”: ”3504175fe0000000000043a7”,
”name”: ”Deckenlicht”,
”functionID”: 4152,
”productRevision”: 789,
”productID”: 200,
”hwInfo”: ”GE−KM200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 784,
”zoneID”: 1038,
”isPresent”: true,
”lastDiscovered”: ”2012−10−26￿15:36:30”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”1970−01−01￿01:00:00”,
”outputMode”: 22,
”buttonID”: 5,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
]
}
],
”groups”: [
{
”id”: 0,
”name”: ”broadcast”,
”isPresent”: false,
”devices”: [
”3504175fe0000000000182f6”,
17

”3504175fe00000000000439c”,
”3504175fe0000000000043a7”
]
},
{
”id”: 1,
”name”: ”yellow”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000182f6”,
”3504175fe00000000000439c”,
”3504175fe0000000000043a7”
]
},
{
”id”: 2,
”name”: ”gray”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 3,
”name”: ”blue”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 4,
”name”: ”cyan”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 5,
”name”: ”magenta”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000151fd”
]
},
{
”id”: 6,
”name”: ”red”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000042dc”
]
},
{
”id”: 7,
”name”: ”green”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 8,
”name”: ”black”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000151fd”
]
}
]
},
{
”id”: 989,
”name”: ”Wohnen”,
”isPresent”: true,
”floor”: 1,
”devices”: [
{
”id”: ”3504175fe0000000000182f6”,
”name”: ”Regalleuchte”,
”functionID”: 4152,
”productRevision”: 49955,
”productID”: 6344,
”hwInfo”: ”GE−SDS200”,
”meterDSID”: ”3504175fe0000010000003dd”,
18

”busID”: 97,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 22,
”buttonID”: 0,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
],
”modelFeatures”: {
”dontcare”: true,
”blink”: false,
”ledauto”: true
}
},
{
”id”: ”3504175fe00000000000439c”,
”name”: ”Stehlampe”,
”functionID”: 4152,
”productRevision”: 789,
”productID”: 200,
”hwInfo”: ”GE−KM200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 153,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 22,
”buttonID”: 0,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
]
}
],
”groups”: [
{
”id”: 0,
”name”: ”broadcast”,
”isPresent”: false,
”devices”: [
”3504175fe0000000000182f6”,
”3504175fe00000000000439c”
]
},
{
”id”: 1,
”name”: ”yellow”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000182f6”,
”3504175fe00000000000439c”
]
},
{
”id”: 2,
”name”: ”gray”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 3,
”name”: ”blue”,
”isPresent”: true,
”devices”: [ ]
},
{
19

”id”: 4,
”name”: ”cyan”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 5,
”name”: ”magenta”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000151fd”
]
},
{
”id”: 6,
”name”: ”red”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000042dc”
]
},
{
”id”: 7,
”name”: ”green”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 8,
”name”: ”black”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000151fd”
]
}
]
},
{
”id”: 1038,
”name”: ”Schlafen”,
”isPresent”: true,
”floor”: 1,
”devices”: [
{
”id”: ”3504175fe0000000000043a7”,
”name”: ”Deckenlicht”,
”functionID”: 4152,
”productRevision”: 789,
”productID”: 200,
”hwInfo”: ”GE−KM200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 784,
”zoneID”: 1038,
”isPresent”: true,
”lastDiscovered”: ”2012−10−26￿15:36:30”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”1970−01−01￿01:00:00”,
”outputMode”: 22,
”buttonID”: 5,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
]
}
],
”groups”: [
{
”id”: 0,
”name”: ”broadcast”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000043a7”
]
},
{
20

”id”: 1,
”name”: ”yellow”,
”isPresent”: true,
”devices”: [
”3504175fe0000000000043a7”
]
},
{
”id”: 2,
”name”: ”gray”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 3,
”name”: ”blue”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 4,
”name”: ”cyan”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 5,
”name”: ”magenta”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 6,
”name”: ”red”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 7,
”name”: ”green”,
”isPresent”: true,
”devices”: [ ]
},
{
”id”: 8,
”name”: ”black”,
”isPresent”: true,
”devices”: [ ]
}
]
}
],
”clusters”: [
{
”id”: 16,
”name”: ”South￿−￿Class￿1￿−￿7”,
”color”: 2,
”applicationType”: 2,
”isPresent”: true,
”isValid”: true,
”CardinalDirection”: ”south”,
”ProtectionClass”: 1,
”isAutomatic”: true,
”configurationLock”: false,
”devices”: [
”302ed89f43f0000000000d000009f75a00”,
”303505d7f800000000000f40000aa96b00”
],
”lockEvents”: []
},
{
”id”: 17,
”name”: ”Markisen”,
”color”: 2,
”applicationType”: 2,
”isPresent”: true,
”isValid”: true,
”CardinalDirection”: ”none”,
21

”ProtectionClass”: 0,
”isAutomatic”: false,
”configurationLock”: false,
”devices”: [
”302ed89f43f0000000000a00000bece800”
],
”lockEvents”: [
”wind”, ”rain”, ”hail”
]
}
],
”floors”: [
{
”id”: 0, // valid id starts at 0. id −1 is ”no floor” for ”all devices” and ”new devices”
”orderId”: 1,
”name”: ”My￿1st￿floor”,
”zones”: [ 989, 1038 ]
}
]
}
}
}
2.5.2
getDevices
Returns an array containing all devices of the apartment.
Synopsis
HTTP GET /json/apartment/getDevices
Parameter
None
Response
HTTP Status 200
result array of devices
Sample
GET /json/apartment/getDevices
{
”ok”: true,
”result”: [
{
”id”: ”3504175fe0000000000182f6”,
”name”: ”Regalleuchte”,
”functionID”: 4152,
”productRevision”: 49955,
”productID”: 6344,
”hwInfo”: ”GE−SDS200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 97,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 22,
”buttonID”: 0,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
22

]
},
{
”id”: ”3504175fe00000000000439c”,
”name”: ”Stehlampe”,
”functionID”: 4152,
”productRevision”: 789,
”productID”: 200,
”hwInfo”: ”GE−KM200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 153,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 22,
”buttonID”: 0,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
]
},
{
”id”: ”3504175fe0000000000151fd”,
”name”: ”Fernseher”,
”functionID”: 33041,
”productRevision”: 41761,
”productID”: 5320,
”hwInfo”: ”SW−ZWS200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 693,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 39,
”buttonID”: 0,
”buttonActiveGroup”: 5,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”5”,
”8”
]
},
{
”id”: ”3504175fe000000000001234”,
”name”: ”Wandlampe”,
”functionID”: 4144,
”productRevision”: 789,
”productID”: 1234,
”hwInfo”: ”GE−TKM210”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 782,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”2012−10−22￿16:22:02”,
”outputMode”: 22,
”buttonID”: 4,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
]
},
{
”id”: ”3504175fe0000000000043a7”,
”name”: ”Deckenlicht”,
23

”functionID”: 4152,
”productRevision”: 789,
”productID”: 200,
”hwInfo”: ”GE−KM200”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 784,
”zoneID”: 1038,
”isPresent”: true,
”lastDiscovered”: ”2012−10−26￿15:36:30”,
”firstSeen”: ”2012−10−22￿16:22:02”,
”inactiveSince”: ”1970−01−01￿01:00:00”,
”outputMode”: 22,
”buttonID”: 5,
”buttonActiveGroup”: 1,
”buttonInputMode”: 0,
”buttonInputIndex”: 0,
”buttonInputCount”: 1,
”groups”: [
”1”
]
},
{
”id”: ”3504175fe0000000000042dc”,
”name”: ”Paniktaster”,
”functionID”: 24896,
”productRevision”: 790,
”productID”: 1225,
”hwInfo”: ”RT−TKM201”,
”meterDSID”: ”3504175fe0000010000003dd”,
”busID”: 785,
”zoneID”: 989,
”isPresent”: false,
”lastDiscovered”: ”2012−10−24￿11:17:29”,
”firstSeen”: ”2012−10−23￿16:23:38”,
”inactiveSince”: ”2012−10−24￿11:01:40”,
”outputMode”: 0,
”buttonID”: 17,
”buttonActiveGroup”: 154,
”buttonInputMode”: 20,
”buttonInputIndex”: 0,
”buttonInputCount”: 0,
”groups”: [
”6”
]
}
]
}
2.5.3
getCircuits
Returns an array containing all digitalSTROM-Meters of the apartment.
Synopsis
HTTP GET /json/apartment/getCircuits
Parameter
None
Response
HTTP Status 200
result.circuits array of digitalSTROM Meters
Sample
GET /json/apartment/getCircuits
{
24

”ok”: true,
”result”: {
”circuits”: [
{
”name”: ”dSM03DD−#1”,
”dsid”: ”3504175fe0000010000003dd”,
”hwVersion”: 721409,
”armSwVersion”: 17498112,
”dspSwVersion”: 16908800,
”apiVersion”: 517,
”hwName”: ””,
”isPresent”: true,
”isValid”: true
},
{
”name”: ”dSM040E−#2”,
”dsid”: ”3504175fe00000100000040e”,
”hwVersion”: 721409,
”armSwVersion”: 17498112,
”dspSwVersion”: 16908800,
”apiVersion”: 517,
”hwName”: ””,
”isPresent”: true,
”isValid”: true
}
]
}
}
2.5.4
getVdcs
Returns an array containing all vDCs matching the given implementationId in the apartment.
Synopsis
HTTP GET /json/apartment/getVdcs
Parameter
Parameter
Description
Remarks
implementationId implementationId of the vDC Mandatory
Response
HTTP Status 200
result.vdcs array of vDCs
Sample
GET /json/apartment/getVdcs&implementationId=DALI_Device_Container
{
”ok”: true,
”result”: {
”vdcs”: [
{
”name”: ”DALI”,
”dsid”: ””,
”dSUID”: ”8b7dd1d5f2a158b780010dc3dd7f2a4a00”,
”DisplayID”: ”8…b7dd1d5”,
”hwVersion”: 0,
”hwVersionString”: ””,
”swVersion”: ”1.6.0.9”,
25

”armSwVersion”: 0,
”dspSwVersion”: 0,
”apiVersion”: 771,
”hwName”: ”P44−DSB−DEH￿DALI”,
”isPresent”: true,
”isValid”: true,
”busMemberType”: 33,
”hasDevices”: true,
”hasMetering”: false,
”VdcConfigURL”: ”http://172.17.0.77:80”,
”VdcModelUID”: ”1A07F24C6D7758CE8055ADDD65E5087300”,
”VdcHardwareGuid”: ””,
”VdcHardwareModelGuid”: ””,
”VdcImplementationId”: ”DALI_Device_Container”,
”VdcVendorGuid”: ””,
”VdcOemGuid”: ””,
”ignoreActionsFromNewDevices”: false
}
]
}
}
2.5.5
removeMeter
Removes an inactive digitalSTROM-Meter object from the installation.
Synopsis
HTTP GET /json/apartment/removeMeter
Parameter
Parameter Description
Remarks
dsid
dSID of the digitalSTROM-Meter Mandatory
Response
HTTP Status 200
result array of digitalSTROM Meters
Sample
GET /json/apartment/removeMeter?dsid=3504175fe00000100000040e
{
”ok” : true
}
2.6
Sensors
2.6.1
Get Assigned Sensors
Returns the list of assigned sensor devices in all zones.
Synopsis
HTTP GET /json/apartment/getAssignedSensors
Parameter
None
26

Response
HTTP Status 200
id
Id of this zone
name
Name of this zone
sensorType Numerical value of the sensor type
dsuid
dSUID of the source device
Sample
GET /json/apartment/getAssignedSensors
{
”ok”:true,
”result”:
”zones”: [
{
”id”: 1,
”name”: ”Living￿Room”,
”sensors”: [
{
”sensorType”: 9,
”dsuid”: 3504175fe00000000000000000016be700
},
{
”sensorType”: 11,
”dsuid”: 3504175fe00000000000000000016be700
}
]
},
{
”id”: 2,
”name”: ”Kitchen”,
”sensors”: [
{
”sensorType”: 9,
”dsuid”: 3504175fe0000000000000000001456700
}
]
}
]
}
}
2.6.2
Get Sensor Values
Returns a list of sensor relevant for the apartment.
For the apartment the temperature, humidity, and brightness are sensor types that are tracked. Addi-
tionally there is
For each zone the temperature, humidity, CO2 concentration and brightness are sensor types that are
tracked. Typically there is one device as a zone reference for these values.
If there is no standard device defined for a sensor type or if no measurement is available there is neither
the value or time field returned.
Synopsis
HTTP GET /json/apartment/getSensorValues
Parameter
None
27

Response
HTTP Status 200
The result object contains the following outdoor measurements:
temperature
Temperature value and timestamp of last measurement
humidity
Humidity value and timestamp of last measurement
brightness
Brightness value and timestamp of last measurement
precipitation
Precipitation value and timestamp of last measurement
airpressure
Air pressure value and timestamp of last measurement
windspeed
Wind speed value and timestamp of last measurement
winddirection Wind direction in degrees value and timestamp of last measurement
gustspeed
Gust speed value and timestamp of last measurement
gustdirection Gust direction in degrees value and timestamp of last measurement
If there is external weather service data available from my.digitalSTROM for the geo location of the
installation it will be provided as well:
WeatherIconId
WeatherConditionId
WeatherServiceId
WeatherServiceTime
The result object contains a ”zones” field with an array of all zones of the apartment and the relevant
sensor data:
TemperatureValue
Temperature value
TemperatureValueTime
Timestamp of the temperature measurement
HumidityValue
Humidity value
HumidityValueTime
Timestamp of the humidity measurement
CO2ConcentrationValue
CO2Concentration value
CO2ConcentrationValueTime Timestamp of the CO2 concentration measurement
BrightnessValue
Brightness value
BrightnessValueTime
Timestamp of the brightness measurement
Sample
GET /json/apartment/getSensorValues
{
”ok”: true,
”result”: {
”weather”: {
”WeatherIconId”: ”04d”,
”WeatherConditionId”: ”803”,
”WeatherServiceId”: ”7”,
”WeatherServiceTime”: ”2017−03−20T14:33:50.328Z”
},
”outdoor”: {
”temperature”: {
”value”: 20.975,
”time”: ”2017−03−20T13:53:15.603Z”
28

},
”humidity”: {
”value”: 71,
”time”: ”2017−03−20T13:53:15.603Z”
},
”windspeed”: {
”value”: 1,
”time”: ”2017−03−20T14:33:29.946Z”
},
”winddirection”: {
”value”: 0,
”time”: ”2017−03−20T14:33:29.943Z”
},
”gustspeed”: {
”value”: 7.2,
”time”: ”2017−03−20T14:33:29.947Z”
},
”gustdirection”: {
”value”: 0.25,
”time”: ”2017−03−20T14:33:29.947Z”
},
”precipitation”: {
”value”: 0,
”time”: ”2017−03−20T14:33:29.948Z”
},
”airpressure”: {
”value”: 1010,
”time”: ”2017−03−20T14:33:29.761Z”
}
},
”zones”: [
{
”id”: 1142,
”name”: ”Küche”,
”values”: [ ]
},
{
”id”: 1168,
”name”: ”Wohnzimmer”,
”values”: [
{
”TemperatureValue”: 22.55,
”TemperatureValueTime”: ”2014−10−13T18:07:24.528+0200”
},
{
”HumidityValue”: 59.2,
”HumidityValueTime”: ”2014−10−13T18:07:24.638+0200”
},
{
”CO2ConcentrationValue”: 1209.205182943208,
”CO2ConcentrationValueTime”: ”2014−10−13T11:45:53.756+0200”
}
]
},
{
”id”: 1191,
”name”: ”Galerie”,
”values”: [
{
”TemperatureValue”: 21.75,
”TemperatureValueTime”: ”2014−10−13T18:04:13.382+0200”
},
{
”HumidityValue”: 64.4,
”HumidityValueTime”: ”2014−10−13T18:04:13.480+0200”
}
]
},
{
”id”: 1192,
”name”: ”Flur”,
”values”: [
{
”TemperatureValue”: 21.95000000000005,
”TemperatureValueTime”: ”2014−10−13T18:04:10.022+0200”
}
]
},
29

{
”id”: 3,
”name”: ”Wintergarten”,
”values”: [
{
”TemperatureValue”: 19.67500000000001,
”TemperatureValueTime”: ”2014−10−13T18:06:07.659+0200”
},
{
”HumidityValue”: 66,
”HumidityValueTime”: ”2014−10−13T18:06:07.790+0200”
}
]
},
{
”id”: 10,
”name”: ”Terrasse”,
”values”: [ ]
}
]
}
}
2.6.3
Set Weather Information
Synopsis
HTTP GET /json/apartment/setWeatherInformation
Parameter
Parameter Description
Remarks
icon
string id, last character d / n indicates day / night
Mandatory
condition
string id, see https://openweathermap.org/weather-conditions Mandatory
service
Service id
Optional
ts
string timestamp
Optional
Sample
GET /json/apartment/setWeatherInformation?icon=10n&condition=500’
{
”ok”:true
}
2.7
Heating
2.7.1
Get Temperature Control Status
Get the current status of temperature control in all zones.
Synopsis
HTTP GET /json/apartment/getTemperatureControlStatus
Parameter
None
30

Response
HTTP Status 200
id
Id of the zone
name
Name of the zone
ControlMode
Control mode: 0=off; 1=pid-control; 2=zone-follower;
3=fixed-value; 4=manual
OperationMode
Current operation mode of the control
TemperatureValue
Current temperature of the zone
TemperatureValueTime Timestamp of last temperature data update, seconds
since epoch
NominalValue
Target temperature of this zone
NominalValueTime
Timestamp of last set point change, seconds since
epoch
ControlValue
Current control value
ControlValueTime
Timestamp of last control value data update, seconds
since epoch
Sample
GET /json/apartment/getTemperatureControlStatus
{
”ok”: true,
”result”:
”zones”: [
{
”id”: 1,
”name”: ”Living￿Room”,
”ControlMode”: 1,
”OperationMode”: 4,
”TemperatureValue”: 20.7,
”NominalValue”: 20.0,
”ControlValue”: 92.5,
”TemperatureValueTime”: 2014−10−08T18:21:05Z,
”NominalValueTime”: 2014−10−08T18:00:00Z,
”ControlValueTime”: 2014−10−08T18:22:00Z
},
{
”id”: 2,
”name”: ”Kitchen”,
”ControlMode”: 2,
”ControlValue”: 92.5,
”ControlValueTime”: 2014−10−08T18:19:00Z
},
{
”id”: 3,
”name”: ”Corridor”,
”ControlMode”: 3,
”OperationMode”: 2,
”ControlValue”: 80
}
]
}
}
2.7.2
Get Temperature Control Configuration
Get the configuration of the temperature control settings for all zones.
Synopsis
HTTP GET /json/apartment/getTemperatureControlConfig
31

Parameter
None
Response
HTTP Status 200
id
Id of this zone
name
Name of this zone
ControlMode
Control mode: 0=off; 1=pid-control; 2=zone-follower;
3=fixed-offset; 4=manual
ManualValue
Fixed control value for manual mode (mode 4 only)
ReferenceZone
Zone number of the reference zone (mode 2 only)
CtrlOffset
Static control value offset (mode 2 only)
EmergencyValue Fixed control value in case of malfunction (mode 1
only)
CtrlKp
Control proportional factor
CtrlTs
Control sampling time
CtrlTi
Control integrator time constant
CtrlKd
Control differential factor
CtrlImin
Control minimum integrator value
CtrlImax
Control maximum integrator value
CtrlYmin
Control minimum control value
CtrlYmax
Control maximum control value
CtrlAntiWindUp
Control integrator anti wind up: 0=inactive, 1=active
Sample
GET /json/apartment/getTemperatureControlConfig
{
”ok”: true,
”result”:
”zones”: [
{
”id”: 1,
”name”: ”Living￿Room”,
”ControlMode”: 1,
”EmergencyValue”: 50,
”CtrlKp”: 5.2,
”CtrlTs”: 240,
”CtrlTi”: 1,
”CtrlKd”: 1,
”CtrlImin”: 600,
”CtrlImax”: 2400,
”CtrlYmin”: 0,
”CtrlYmax”: 100,
”CtrlAntiWindUp”: 1
},
{
”id”: 2,
”name”: ”Kitchen”,
”ControlMode”: 2,
”ReferenceZone”: 0,
”CtrlOffset”: −10
},
...
{
”id”: 3,
”name”: ”Corridor”,
”ControlMode”: 3
32

}
]
}
}
2.7.3
Get Temperature Control Values
Returns a list of all temperature control preset values of all zones. Every control operation mode has
up to 15 presets defined, where 6 of them are actually used by the system.
Synopsis
HTTP GET /json/apartment/getTemperatureControlValues
Parameter
None
Response
HTTP Status 200
id
Id of this zone
name
Name of this zone
Off
Preset value for operation mode 0: ”Off”
Comfort
Preset value for operation mode 1: ”Comfort”
Economy
Preset value for operation mode 2: ”Economy”
NotUsed
Preset value for operation mode 3: ”Not Used”
Night
Preset value for operation mode 4: ”Night”
Holiday
Preset value for operation mode 5: ”Holiday”
Cooling
Preset value for operation mode 6: ”Cooling”
CoolingOff Preset value for operation mode 7: ”CoolingOff”
Sample
GET /json/apartment/getTemperatureControlValues
{
”ok”: true,
”result”:
”zones”: [
{
”id”: 1,
”name”: ”Living￿Room”,
”Off”: 6,
”Comfort”: 21,
”Economy”: 20,
”NotUsed”: 18,
”Night”: 16,
”Holiday”: 12,
”Cooling”: 23,
”CoolingOff”: 50,
},
...
{
”id”: 972,
”name”: ””,
”Off”: 8,
”Comfort”: 22,
”Economy”: 20,
”NotUsed”: 18,
”Night”: 17,
”Holiday”: 16,
33

”Cooling”: 23,
”CoolingOff”: 50
}
}
}
2.7.4
Get Temperature Control Configuration v2
Get the temperature control configuration parameters for each zone with one call.
Synopsis
HTTP GET /json/apartment/getTemperatureControlConfig2
Parameter
None
Response
HTTP Status 200
id
Id of this zone
name
Name of this zone
mode
Current Control Mode of the zone: ”off”, ”control”, ”zoneFollower”, ”fixed”, ”manual”
targetTemperatures Set point temperatures for each operation mode of the zone
fixedValues
Fixed control values for each operation mode of the zone
controlMode
Object with the PID controller related parameters
zoneFollowerMode
Object with the zone follower related parameters
manualMode
Object with the manual mode parameter control value
controlMode
emergencyValue Fixed control value in case of malfunction
ctrlKp
Control proportional factor
ctrlTs
Control sampling time
ctrlTi
Control integrator time constant
ctrlKd
Control differential factor
ctrlImin
Control minimum integrator value
ctrlImax
Control maximum integrator value
ctrlYmin
Control minimum control value
ctrlYmax
Control maximum control value
ctrlAntiWindUp
Control integrator anti wind up
zoneFollowerMode
referenceZone Zone number of the reference zone
ctrlOffset
Control value offset
manualMode
controlValue Control value for manual mode
34

Sample
GET /json/apartment/getTemperatureControlConfig2?id=1237
{
”ok”: true,
”result”: {
”zones”: {
”1” : {
”name”: ”Living￿room”,
”config”: {
”mode” : ”manual”,
”targetTemperatures” : {
”0”: 6, ”1”: 23.5, ”2”: 22, ”3”: 19,
”4”: 18, ”5”: 18, ”6”: 22, ”7”: 50,
”8”: 24, ”9”: 28, ”10”: 32, ”11”: 30
},
”fixedValues” : {
”0”: 0, ”1”: 100, ”2”: 90, ”3”: 80,
”4”: 70, ”5”: 25, ”6”: 100, ”7”: 0,
”8”: 80, ”9”: 60, ”10”: 40, ”11”: 25
},
”controlMode” : {
”emergencyValue”: 50,
”ctrlKp”: 5.2,
”ctrlTs”: 240,
”ctrlTi”: 1,
”ctrlKd”: 1,
”ctrlImin”: 600,
”ctrlImax”: 2400,
”ctrlYmin”: 0,
”ctrlYmax”: 100,
”ctrlAntiWindUp”: 1
},
”zoneFollowerMode” : {
”referenceZone”: 38523,
”ctrlOffset”: 10
},
”manualMode” : {
”controlValue” : 30
}
}
}
}
}
}
2.8
DevicesFirstSeen
2.8.1
setDevicesFirstSeen
Sets the FirstSeen property of all devices with first seen date before 1 January 2011. All other devices
are not modified
Synopsis
HTTP GET /json/apartment/setDevicesFirstSeen
Parameter Description
Remarks
time
ISO8601 time when the devices were registered Mandatory
Parameter
Response
HTTP Status 200
ok true
35

Sample
GET /json/apartment/setDevicesFirstSeen?time=2012−06−14T10:42:13Z
{
”ok”:true
}
2.9
ModelFeatures
ModelFeatures are used to determine the visibility and (to some extent) the functionality of the Configurator-
UI.
2.9.1
getModelFeatures
Returns the known ModelFeatures.
Synopsis
HTTP GET /json/apartment/getModelFeatures
Parameter Description Remarks
Parameter
Response
HTTP Status 200
result.modelFeatures
object containing the known model
features. The features are ordered
according to the device’s color (e.g.
“GE”, “SW”, etc.).
Always the
most specific model feature ap-
plies: e.g. (refer to the example be-
low) “KM:200” from “GE” applies to
a GE-KM220 device; while a “KM:2”
from “GE” applies to a GE-KM210
device.
result.modelFeatures.<color>
result.modelFeatures.<color>.<model>
reference
object containing all defined model
features
Sample
GET /json/apartment/getModelFeatures
{
”ok”: true,
”result”: {
”modelFeatures”: {
”GE”: {
”KM:220”: {
”dontcare”: true,
”blink”: true,
”ledauto”: true
},
”KM:2”: {
”dontcare”: true,
36

”blink”: true,
”ledauto”: true
},
”KL:200”: {
”dontcare”: true,
”blink”: true
}
},
”GR”: {
”KL:210”: {
”dontcare”: true,
”blink”: true,
”ledauto”: true
}
”KL:2”: {
”dontcare”: true,
”blink”: true
}
}
},
”reference”: {
”dontcare”: false,
”blink”: false,
”ledauto”: false,
”leddark”: false
}
}
}
2.10
MuteFire
Mutes the fire alarm. Does nothing if fire alarm is not active or if fire mute is not enabled in digitalSTROM-
Server.
See also fire, fireMuteEnabled states in the system-interfaces document.
Synopsis
HTTP GET /json/apartment/muteFire
Response
HTTP Status 200
ok true
Sample
GET /json/apartment/muteFire
{
”ok”:true
}
2.11
Controller Statuses
2.11.1
getControllerStatus
Polls status of all controllers in the system.
Synopsis
HTTP GET /json/apartment/getControllerStatuses
37

Parameter
Parameter Description
Remarks
lang
Locale Code based on Language_Region pattern, e.g. en_EN, de_DE
Response
HTTP Status 200
code
off | ready | error
message Translated message for user providing further details
Sample
GET /json/apartment/getControllerStatuses
{
”ok”:true,
”result” : {
”controllerStatuses”:{
”302ed89f43f0000000000e400000605700”:{”code”:”ready”,”message”:”Ready”},
”3504175fe0000000000000100000071700”:{”code”:”ready”,”message”:”Ready”},
”7f40dee794044465a38bcdf7457094e300”:{”code”:”ready”,”message”:”Authorization￿will￿expire￿in￿1￿week.”}
}
}
}
38

