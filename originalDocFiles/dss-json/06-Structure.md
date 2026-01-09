6 Structure
===========

6
Structure
6.1
Zone
6.1.1
addZone
Adds a zone with the given Id. The zone is added to the digitalSTROM-Server data model only and initially
does not have any devices associated.
Synopsis
HTTP GET /json/structure/addZone
Parameter
Parameter Description
zoneID
unique numerical identifier for the new zone
floorID
ID of the floor in which the zone will be created. If not specified, the default floor will be assigned
Response
HTTP Status 200
ok
true
result.zoneID numeric identifier for the new zone
Sample
GET /json/structure/addZone?zoneID=1&floorID=34
{
”ok”:true,
”result”:
{
”zoneID”:1
}
}
6.1.2
removeZone
Removes the zone with the give Id from the installation. A zone can only be removed if it has no asso-
ciated devices.
Synopsis
HTTP GET /json/structure/removeZone
Parameter Description
Remarks
zoneID
unique numerical identifier Mandatory
Parameter
Response
HTTP Status 200
125

ok true
Sample
GET /json/structure/removeZone?zoneID=1234
{
”ok”:true
}
6.1.3
floorAddZone
Associates a zone with a new floor. The zone is automatically removed from the old floor.
Synopsis
HTTP GET /json/structure/floorAddZone
Parameter
Parameter Description
Remarks
zoneID
ID of the zone to move Mandatory
floorID
ID for the new floor
Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/floorAddZone?zoneID=1234&floorID=23
{
”ok”:true
}
6.2
Floor
6.2.1
addFloor
Adds a floor.
Synopsis
HTTP GET /json/structure/addFloor
Response
HTTP Status 200
ok
true
result.floorID
numeric identifier for the new floor
result.orderID numeric identifier for sorting the floor in UIs
126

Sample
GET /json/structure/addFloor
{
”ok”:true,
”result”:
{
”floorID”:1,
”orderID”:3
}
}
6.2.2
removeFloor
Removes the floor with the give Id from the installation. A floor can only be removed if it has no asso-
ciated zones.
Synopsis
HTTP GET /json/structure/removeFloor
Parameter Description
Remarks
floorID
unique numerical identifier Mandatory
Parameter
Response
HTTP Status 200
ok true
Sample
GET /json/structure/removeFloor?floorID=1234
{
”ok”:true
}
6.2.3
setFloorName
Sets the floor name.
Synopsis
HTTP GET /json/zone/setFloorName
Parameter Description
Remarks
name
identifier string for the floor Mandatory
Parameter
Response
HTTP Status 200
127

ok true
Sample
GET /json/zone/setFloorName?id=1237&name=”1st￿floor”
{
”ok”:true
}
6.3
Group
6.3.1
addGroup
Adds a user group to the zone with the given Id.
Synopsis
HTTP GET /json/structure/addGroup
Parameter
Parameter
Description
Remarks
zoneID
identifier of the zone where the group has to be
created
Mandatory
groupID
numerical identifier for the new group
Mandatory
if groupAu-
toSelect
is
not given
groupAutoSelect flag to let the system find a free group id
Mandatory
if
groupID
is not given
groupColor
application state machine selector for the new
group, default is none
Optional
groupName
name for the new group
Optional
Response
HTTP Status 200
result.groupID
numeric identifier for the new group
result.zoneID
numeric identifier for the zone
result.groupName string, name of the new group
result.groupColor
numeric identifier, color of the new group
Sample
GET /json/structure/addGroup?zoneID=1234&groupAutoSelect=global&groupColor=5&groupName=test
{
”ok”:true,
”result”:
{
”groupID”:41,
128

”zoneID”:1234,
”groupName”:”test”,
”groupColor”:5
}
}
6.3.2
removeGroup
Removes a user group to the zone with the given Id.
Synopsis
HTTP GET /json/structure/removeGroup
Parameter
Parameter Description
Remarks
zoneID
unique numerical identifier for the zone Mandatory
groupID
numerical identifier for the group
Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/removeGroup?zoneID=1234&groupID=42
{
”ok”:true
}
6.3.3
groupSetName
Rename a group.
Synopsis
HTTP GET /json/structure/groupSetName
Parameter
Parameter Description
Remarks
zoneID
unique numerical identifier for the zone Mandatory
groupID
numerical identifier for the group
Mandatory
newName
string, new name for the group
Mandatory
Response
HTTP Status 200
129

ok true
Sample
GET /json/structure/groupSetName?zoneID=1234&groupID=42&newName=test
{
”ok”:true
}
6.3.4
groupSetColor
Change application type of the zone user group or apartment user application.
Synopsis
HTTP GET /json/structure/groupSetColor
Parameter
Parameter Description
Remarks
zoneID
unique numerical identifier for the zone
Mandatory
groupID
numerical identifier for the group
Mandatory
newColor
numerical identifier of the application type the group Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/groupSetColor?zoneID=1234&groupID=42&newColor=4
{
”ok”:true
}
6.3.5
groupSetConfiguration
Set application specific attributes for a group. The following attributes are supported:
Attribute
Application
Remarks
activeBasicScenes Ventilation, Recirculation Configure the available levels for ventilation groups
Synopsis
HTTP GET /json/structure/groupSetConfiguration
Parameter
130

Parameter
Description
Remarks
zoneID
unique numerical identifier for the zone
Mandatory
groupID
numerical identifier for the group
Mandatory
configuration string with encoded json object defining the attributes Mandatory
ok true
Response
HTTP Status 200
Sample
GET /json/structure/groupSetConfiguration?zoneID=1234&groupID=10&configuration={”activeBasicScenes”:[0,5,17]}
{
”ok”:true
}
6.3.6
groupGetConfiguration
Get application specific attributes for a group.
Synopsis
HTTP GET /json/structure/groupSetConfiguration
Parameter
Parameter
Description
Remarks
zoneID
unique numerical identifier for the zone
Mandatory
groupID
numerical identifier for the group
Mandatory
configuration string with encoded json object defining the attributes Mandatory
Response
HTTP Status 200
ok
true
result.activeBasicScenes array of basic ventilation scene numbers, e.g. levels (optional)
Sample
GET /json/structure/groupGetConfiguration?zoneID=1234&groupID=10
{
”result”: {
”activeBasicScenes”: [
0,
5,
17,
18,
19,
36
]
131

},
”ok”: true
}
6.4
Cluster
6.4.1
addCluster
Adds a cluster with the given name and color and returns the automatically chosen Id.
Synopsis
HTTP GET /json/structure/addCluster
Parameter
Parameter Description
Remarks
color
application state machine selector for the new
cluster
Mandatory
name
name for the new group
Mandatory
Response
HTTP Status 200
result.clusterID numeric identifier for the new cluster
result.name
string, name of the new cluster
result.color
numeric identifier, color of the new cluster
Sample
GET /json/structure/addCluster?color=2&name=test
{
”ok”:true,
”result”:
{
”clusterID”: 34,
”name”:”test”,
”color”: 2
}
}
6.4.2
removeCluster
Removes a cluster with the given Id.
Synopsis
HTTP GET /json/structure/removeCluster
Parameter
132

Parameter Description
Remarks
clusterID
numerical identifier for the cluster Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/removeCluster?clusterID=34
{
”ok”:true
}
6.4.3
clusterSetName
Rename a cluster.
Synopsis
HTTP GET /json/structure/clusterSetName
Parameter
Parameter Description
Remarks
clusterID
numerical identifier for the cluster Mandatory
newName
string, new name for the cluster
Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/clusterSetName?clusterID=18&newName=test
{
”ok”:true
}
6.4.4
clusterSetColor
Change color of the cluster.
Synopsis
HTTP GET /json/structure/clusterSetColor
Parameter
133

Parameter Description
Remarks
clusterID
numerical identifier for the cluster
Mandatory
newColor
numerical identifier of the color for the cluster Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/clusterSetColor?clusterID=18&newColor=4
{
”ok”:true
}
6.4.5
clusterSetConfigLock
Locks or unlocks the configuration of the cluster. If locked changes of the target application and devices
are not allowed.
Synopsis
HTTP GET /json/structure/clusterSetConfigLock
Parameter
Parameter Description
Remarks
clusterID
numerical identifier for the cluster
Mandatory
lock
numerical value: 0 = unlocked, 1 = locked Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/clusterSetConfigLock?clusterID=18&lock=1
{
”ok”:true
}
6.5
Device
6.5.1
zoneAddDevice
Associates a device with a new zone. A device is automatically removed from the old zone. Only active
devices can be moved to a new zone because the zone configuration has to be synchronized with the
device itself.
134

Synopsis
HTTP GET /json/structure/zoneAddDevice
Parameter
Parameter Description
Remarks
deviceID
DSID of the device to move
Mandatory
zone
unique numerical identifier for the new zone Mandatory
Response
HTTP Status 200
movedDevices array of devices which have been moved
In the case of a failure various different error messages may occur.
Sample
GET /json/structure/zoneAddDevice?deviceID= &zone=1
{
ok: true
result: {
movedDevices: [
{
id: ”3504175fe000000000005854”
name: ””
functionID: 4144
productRevision: 788
productID: 1234
hwInfo: ”GE−TKM210”
meterDSID: ”3504175fe0000010000004d9”
busID: 241
zoneID: 1
isPresent: true
lastDiscovered: ”2012−11−22￿10:35:05”
firstSeen: ”2012−11−19￿14:34:02”
inactiveSince: ”1970−01−01￿01:00:00”
outputMode: 16
buttonID: 12
buttonActiveGroup: 1
buttonInputMode: 0
buttonInputIndex: 0
buttonInputCount: 1
groups: [
”1”
]
}
]
}
}
6.5.2
removeDevice
Removes a device from the data model. Only inactive devices can be removed.
Synopsis
HTTP GET /json/structure/removeDevice
Parameter
135

Parameter Description
Remarks
deviceID
DSID of the device to be removed Mandatory
Response
HTTP Status 200
ok true
Sample
GET /json/structure/removeDevice?deviceID=3504175fe000000000005854
{
ok: false
message: ”Cannot￿remove￿present￿device”
}
6.5.3
groupAddDevice
Adds a device to the user group. Only active devices can be added to additional groups.
Synopsis
HTTP GET /json/structure/groupAddDevice
Parameter
Parameter Description
Remarks
deviceID
DSID of the device
Mandatory if no dsuid
dsuid
DSUID of the device
Mandatory if no deviceID
groupID
unique numerical group identifier Mandatory
Response
HTTP Status 200
ok
true
action
either update or none
devices list of devices that have been changed, only existing if action is ”update”
Sample
GET /json/structure/groupAddDevice?deviceID=3504175fe000000000005854&groupID=42
{
ok: true,
result: {
action: update,
devices: [
{
id: ”3504175fe000000000005854”,
name: ””,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo: ”GE−TKM210”,
136

meterDSID: ”3504175fe0000010000004d9”,
busID: 241,
zoneID: 1,
isPresent: true,
lastDiscovered: ”2012−11−22￿10:35:05”,
firstSeen: ”2012−11−19￿14:34:02”,
inactiveSince: ”1970−01−01￿01:00:00”,
outputMode: 16,
buttonID: 12,
buttonActiveGroup: 1,
buttonInputMode: 0,
buttonInputIndex: 0,
buttonInputCount: 1,
groups: [
”1”, ”42”
]
}
]
}
}
6.5.4
groupRemoveDevice
Removes a device from the user group. Only active devices can be removed from groups.
Synopsis
HTTP GET /json/structure/groupRemoveDevice
Parameter
Parameter Description
Remarks
deviceID
DSID of the device
Mandatory if no dsuid
dsuid
DSUID of the device
Mandatory if no deviceID
groupID
unique numerical group identifier Mandatory
Response
HTTP Status 200
ok
true
action
either update or none
devices list of devices that have been changed, only existing if action is ”update”
Sample
GET /json/structure/groupRemoveDevice?deviceID=3504175fe000000000005854&groupID=42
{
ok: true,
result: {
action: update,
devices: [
{
id: ”3504175fe000000000005854”,
name: ””,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo: ”GE−TKM210”,
meterDSID: ”3504175fe0000010000004d9”,
busID: 241,
137

zoneID: 1,
isPresent: true,
lastDiscovered: ”2012−11−22￿10:35:05”,
firstSeen: ”2012−11−19￿14:34:02”,
inactiveSince: ”1970−01−01￿01:00:00”,
outputMode: 16,
buttonID: 12,
buttonActiveGroup: 1,
buttonInputMode: 0,
buttonInputIndex: 0,
buttonInputCount: 1,
groups: [
”1”
]
}
]
}
}
6.5.5
clusterAddDevice
Adds a device to the cluster. Only active devices can be added to additional cluster.
Synopsis
HTTP GET /json/structure/clusterAddDevice
Parameter
Parameter Description
Remarks
deviceID
DSID of the device
Mandatory if no dsuid
dsuid
DSUID of the device
Mandatory if no deviceID
clusterID
unique numerical cluster identifier Mandatory
Response
HTTP Status 200
ok
true
action
either update or none
devices list of devices that have been changed, only existing if action is ”update”
Sample
GET /json/structure/clusterAddDevice?deviceID=3504175fe000000000005854&groupID=16
{
ok: true,
result: {
action: update,
devices: [
{
id: ”3504175fe000000000005854”,
name: ””,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo: ”GE−TKM210”,
meterDSID: ”3504175fe0000010000004d9”,
busID: 241,
zoneID: 1,
isPresent: true,
138

lastDiscovered: ”2012−11−22￿10:35:05”,
firstSeen: ”2012−11−19￿14:34:02”,
inactiveSince: ”1970−01−01￿01:00:00”,
outputMode: 16,
buttonID: 12,
buttonActiveGroup: 1,
buttonInputMode: 0,
buttonInputIndex: 0,
buttonInputCount: 1,
groups: [
”1”, ”16”
]
}
]
}
}
6.5.6
clusterRemoveDevice
Removes a device from the cluster. Only active devices can be removed from cluster.
Synopsis
HTTP GET /json/structure/clusterRemoveDevice
Parameter
Parameter Description
Remarks
deviceID
DSID of the device
Mandatory if no dsuid
dsuid
DSUID of the device
Mandatory if no deviceID
clusterID
unique numerical cluster identifier Mandatory
Response
HTTP Status 200
ok
true
action
either update or none
devices list of devices that have been changed, only existing if action is ”update”
Sample
GET /json/structure/clusterRemoveDevice?deviceID=3504175fe000000000005854&groupID=16
{
ok: true,
result: {
action: update,
devices: [
{
id: ”3504175fe000000000005854”,
name: ””,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo: ”GE−TKM210”,
meterDSID: ”3504175fe0000010000004d9”,
busID: 241,
zoneID: 1,
isPresent: true,
lastDiscovered: ”2012−11−22￿10:35:05”,
firstSeen: ”2012−11−19￿14:34:02”,
139

inactiveSince: ”1970−01−01￿01:00:00”,
outputMode: 16,
buttonID: 12,
buttonActiveGroup: 1,
buttonInputMode: 0,
buttonInputIndex: 0,
buttonInputCount: 1,
groups: [
”1”
]
}
]
}
}
140

