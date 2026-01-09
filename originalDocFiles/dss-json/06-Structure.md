6 Structure
===========

Structure

6.1
---

Zone

6.1.1
-----

addZone

Adds a zone with the given Id. The zone is added to the digitalSTROM-Server data model only and initially

does not have any devices associated.

**Synopsis**

```json
HTTP GET /json/structure/addZone
```

| Parameter | Description | Remarks |
|---|---|---|
| zoneID | uniquenumericalidentifierforthenewzone | Optional |
| floorID | IDofthefloorinwhichthezonewillbecreated. Ifnotspecified,thedefaultfloorwillbeassigned | Optional |

ID of the floor in which the zone will be created. If not specified, the default floor will be assigned

**Response**

```json
HTTP Status 200
```

result.zoneID numeric identifier for the new zone

**Sample**

```json
GET /json/structure/addZone?zoneID=1&floorID=34
{
”ok” : true ,
”result” :
{
”zoneID” :1
}
}
```

6.1.2
-----

removeZone

Removes the zone with the give Id from the installation. A zone can only be removed if it has no asso-

ciated devices.

**Synopsis**

```json
HTTP GET /json/structure/removeZone
```

| ok result.zoneID | true numericidentifierforthenewzone |
|---|---|

unique numerical identifier Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/removeZone?zoneID=1234
{
”ok” : true
}
```

6.1.3
-----

floorAddZone

Associates a zone with a new floor. The zone is automatically removed from the old floor.

**Synopsis**

```json
HTTP GET /json/structure/floorAddZone
```

| ok | true |
|---|---|

ID of the zone to move Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/floorAddZone?zoneID=1234&floorID=23
{
”ok” : true
}
```

6.2
---

Floor

6.2.1
-----

addFloor

Adds a floor.

**Synopsis**

```json
HTTP GET /json/structure/addFloor
```

**Response**

```json
HTTP Status 200
```

result.orderID numeric identifier for sorting the floor in UIs

**Sample**

```json
GET /json/structure/addFloor
{
”ok” : true ,
”result” :
{
”floorID” :1,
”orderID” :3
}
}
```

6.2.2
-----

removeFloor

Removes the floor with the give Id from the installation. A floor can only be removed if it has no asso-

ciated zones.

**Synopsis**

```json
HTTP GET /json/structure/removeFloor
```

| Parameter | Description | Remarks |
|---|---|---|
| floorID | uniquenumericalidentifier | Mandatory |

unique numerical identifier Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/removeFloor?floorID=1234
{
”ok” : true
}
```

6.2.3
-----

setFloorName

Sets the floor name.

**Synopsis**

```json
HTTP GET /json/zone/setFloorName
```

| ok | true |
|---|---|

identifier string for the floor Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setFloorName?id=1237&name= ”1st￿floor”
{
”ok” : true
}
```

6.3
---

Group

6.3.1
-----

addGroup

Adds a user group to the zone with the given Id.

**Synopsis**

```json
HTTP GET /json/structure/addGroup
```

if groupAu-

groupAutoSelect flag to let the system find a free group id

application state machine selector for the new

**Response**

```json
HTTP Status 200
```

result.groupName string, name of the new group

**Sample**

```json
GET /json/structure/addGroup?zoneID=1234&groupAutoSelect=global&groupColor=5&groupName=test
{
”ok” : true ,
”result” :
{
”groupID” :41,
```

```json
”zoneID” :1234,
”groupName” : ”test” ,
”groupColor” :5
}
}
```

6.3.2
-----

removeGroup

Removes a user group to the zone with the given Id.

**Synopsis**

```json
HTTP GET /json/structure/removeGroup
```

| Parameter | Description | Remarks |
|---|---|---|
| zoneID | uniquenumericalidentifierforthezone | Mandatory |
| groupID | numericalidentifierforthegroup | Mandatory |

unique numerical identifier for the zone Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/removeGroup?zoneID=1234&groupID=42
{
”ok” : true
}
```

6.3.3
-----

groupSetName

Rename a group.

**Synopsis**

```json
HTTP GET /json/structure/groupSetName
```

| ok | true |
|---|---|

unique numerical identifier for the zone Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/groupSetName?zoneID=1234&groupID=42&newName=test
{
”ok” : true
}
```

6.3.4
-----

groupSetColor

Change application type of the zone user group or apartment user application.

**Synopsis**

```json
HTTP GET /json/structure/groupSetColor
```

| ok | true |
|---|---|

numerical identifier of the application type the group Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/groupSetColor?zoneID=1234&groupID=42&newColor=4
{
”ok” : true
}
```

6.3.5
-----

groupSetConfiguration

Set application specific attributes for a group. The following attributes are supported:

activeBasicScenes Ventilation, Recirculation Configure the available levels for ventilation groups

**Synopsis**

```json
HTTP GET /json/structure/groupSetConfiguration
```

configuration string with encoded json object defining the attributes Mandatory

ok true

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/structure/groupSetConfiguration?zoneID=1234&groupID=10&configuration={ ”activeBasicScenes” :[0,5,17]}
{
”ok” : true
}
```

6.3.6
-----

groupGetConfiguration

Get application specific attributes for a group.

**Synopsis**

```json
HTTP GET /json/structure/groupSetConfiguration
```

configuration string with encoded json object defining the attributes Mandatory

**Response**

```json
HTTP Status 200
```

result.activeBasicScenes array of basic ventilation scene numbers, e.g. levels (optional)

**Sample**

```json
GET /json/structure/groupGetConfiguration?zoneID=1234&groupID=10
{
”result” : {
”activeBasicScenes” : [
0,
5,
17,
18,
19,
]
```

```json
},
”ok” :  true
}
```

6.4
---

Cluster

6.4.1
-----

addCluster

Adds a cluster with the given name and color and returns the automatically chosen Id.

**Synopsis**

```json
HTTP GET /json/structure/addCluster
```

| Parameter | Description | Remarks |
|---|---|---|
| color | application state machine selector for the new | Mandatory |
| name | cluster | Mandatory |
|  | nameforthenewgroup |  |

application state machine selector for the new

**Response**

```json
HTTP Status 200
```

result.clusterID numeric identifier for the new cluster

**Sample**

```json
GET /json/structure/addCluster?color=2&name=test
{
”ok” : true ,
”result” :
{
”clusterID” : 34,
”name” : ”test” ,
”color” : 2
}
}
```

6.4.2
-----

removeCluster

Removes a cluster with the given Id.

**Synopsis**

```json
HTTP GET /json/structure/removeCluster
```

| Parameter | Description | Remarks |
|---|---|---|
| clusterID | numericalidentifierforthecluster | Mandatory |

numerical identifier for the cluster Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/removeCluster?clusterID=34
{
”ok” : true
}
```

6.4.3
-----

clusterSetName

Rename a cluster.

**Synopsis**

```json
HTTP GET /json/structure/clusterSetName
```

| ok | true |
|---|---|

numerical identifier for the cluster Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/clusterSetName?clusterID=18&newName=test
{
”ok” : true
}
```

6.4.4
-----

clusterSetColor

Change color of the cluster.

**Synopsis**

```json
HTTP GET /json/structure/clusterSetColor
```

| Parameter | Description | Remarks |
|---|---|---|
| clusterID | numericalidentifierforthecluster | Mandatory |
| newColor | numericalidentifierofthecolorforthecluster | Mandatory |

numerical identifier of the color for the cluster Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/clusterSetColor?clusterID=18&newColor=4
{
”ok” : true
}
```

6.4.5
-----

clusterSetConfigLock

Locks or unlocks the configuration of the cluster. If locked changes of the target application and devices

are not allowed.

**Synopsis**

```json
HTTP GET /json/structure/clusterSetConfigLock
```

| ok | true |
|---|---|

numerical value: 0 = unlocked, 1 = locked Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/clusterSetConfigLock?clusterID=18&lock=1
{
”ok” : true
}
```

6.5
---

Device

6.5.1
-----

zoneAddDevice

Associates a device with a new zone. A device is automatically removed from the old zone. Only active

devices can be moved to a new zone because the zone configuration has to be synchronized with the

device itself.

**Synopsis**

```json
HTTP GET /json/structure/zoneAddDevice
```

| Parameter | Description | Remarks |
|---|---|---|
| deviceID | DSIDofthedevicetomove | Mandatory |
| zone | uniquenumericalidentifierforthenewzone | Mandatory |

unique numerical identifier for the new zone Mandatory

**Response**

```json
HTTP Status 200
```

movedDevices array of devices which have been moved

In the case of a failure various different error messages may occur.

**Sample**

```json
GET /json/structure/zoneAddDevice?deviceID= &zone=1
{
ok:  true
result: {
movedDevices: [
{
id:  ”3504175fe000000000005854”
name:  ””
functionID: 4144
productRevision: 788
productID: 1234
hwInfo:  ”GE − TKM210”
meterDSID:  ”3504175fe0000010000004d9”
busID: 241
zoneID: 1
isPresent:  true
lastDiscovered:  ”2012 − 11 − 22￿10:35:05”
firstSeen:  ”2012 − 11 − 19￿14:34:02”
inactiveSince:  ”1970 − 01 − 01￿01:00:00”
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
```

6.5.2
-----

removeDevice

Removes a device from the data model. Only inactive devices can be removed.

**Synopsis**

```json
HTTP GET /json/structure/removeDevice
```

| Parameter | Description | Remarks |
|---|---|---|
| deviceID | DSIDofthedevicetoberemoved | Mandatory |

DSID of the device to be removed Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/structure/removeDevice?deviceID=3504175fe000000000005854
{
ok:  false
message:  ”Cannot￿remove￿present￿device”
}
```

6.5.3
-----

groupAddDevice

Adds a device to the user group. Only active devices can be added to additional groups.

**Synopsis**

```json
HTTP GET /json/structure/groupAddDevice
```

| ok | true |
|---|---|

unique numerical group identifier Mandatory

**Response**

```json
HTTP Status 200
```

devices list of devices that have been changed, only existing if action is ”update”

**Sample**

```json
GET /json/structure/groupAddDevice?deviceID=3504175fe000000000005854&groupID=42
{
ok:  true ,
result: {
action: update,
devices: [
{
id:  ”3504175fe000000000005854” ,
name:  ”” ,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo:  ”GE − TKM210” ,
```

```json
meterDSID:  ”3504175fe0000010000004d9” ,
busID: 241,
zoneID: 1,
isPresent:  true ,
lastDiscovered:  ”2012 − 11 − 22￿10:35:05” ,
firstSeen:  ”2012 − 11 − 19￿14:34:02” ,
inactiveSince:  ”1970 − 01 − 01￿01:00:00” ,
outputMode: 16,
buttonID: 12,
buttonActiveGroup: 1,
buttonInputMode: 0,
buttonInputIndex: 0,
buttonInputCount: 1,
groups: [
”1” ,  ”42”
]
}
]
}
}
```

6.5.4
-----

groupRemoveDevice

Removes a device from the user group. Only active devices can be removed from groups.

**Synopsis**

```json
HTTP GET /json/structure/groupRemoveDevice
```

| Parameter | Description | Remarks |
|---|---|---|
| deviceID | DSIDofthedevice | Mandatoryifnodsuid |
| dsuid | DSUIDofthedevice | MandatoryifnodeviceID |
| groupID | uniquenumericalgroupidentifier | Mandatory |

unique numerical group identifier Mandatory

**Response**

```json
HTTP Status 200
```

devices list of devices that have been changed, only existing if action is ”update”

**Sample**

```json
GET /json/structure/groupRemoveDevice?deviceID=3504175fe000000000005854&groupID=42
{
ok:  true ,
result: {
action: update,
devices: [
{
id:  ”3504175fe000000000005854” ,
name:  ”” ,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo:  ”GE − TKM210” ,
meterDSID:  ”3504175fe0000010000004d9” ,
busID: 241,
```

```json
zoneID: 1,
isPresent:  true ,
lastDiscovered:  ”2012 − 11 − 22￿10:35:05” ,
firstSeen:  ”2012 − 11 − 19￿14:34:02” ,
inactiveSince:  ”1970 − 01 − 01￿01:00:00” ,
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
```

6.5.5
-----

clusterAddDevice

Adds a device to the cluster. Only active devices can be added to additional cluster.

**Synopsis**

```json
HTTP GET /json/structure/clusterAddDevice
```

| Parameter | Description | Remarks |
|---|---|---|
| deviceID | DSIDofthedevice | Mandatoryifnodsuid |
| dsuid | DSUIDofthedevice | MandatoryifnodeviceID |
| clusterID | uniquenumericalclusteridentifier | Mandatory |

unique numerical cluster identifier Mandatory

**Response**

```json
HTTP Status 200
```

devices list of devices that have been changed, only existing if action is ”update”

**Sample**

```json
GET /json/structure/clusterAddDevice?deviceID=3504175fe000000000005854&groupID=16
{
ok:  true ,
result: {
action: update,
devices: [
{
id:  ”3504175fe000000000005854” ,
name:  ”” ,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo:  ”GE − TKM210” ,
meterDSID:  ”3504175fe0000010000004d9” ,
busID: 241,
zoneID: 1,
isPresent:  true ,
```

```json
lastDiscovered:  ”2012 − 11 − 22￿10:35:05” ,
firstSeen:  ”2012 − 11 − 19￿14:34:02” ,
inactiveSince:  ”1970 − 01 − 01￿01:00:00” ,
outputMode: 16,
buttonID: 12,
buttonActiveGroup: 1,
buttonInputMode: 0,
buttonInputIndex: 0,
buttonInputCount: 1,
groups: [
”1” ,  ”16”
]
}
]
}
}
```

6.5.6
-----

clusterRemoveDevice

Removes a device from the cluster. Only active devices can be removed from cluster.

**Synopsis**

```json
HTTP GET /json/structure/clusterRemoveDevice
```

| Parameter | Description | Remarks |
|---|---|---|
| deviceID | DSIDofthedevice | Mandatoryifnodsuid |
| dsuid | DSUIDofthedevice | MandatoryifnodeviceID |
| clusterID | uniquenumericalclusteridentifier | Mandatory |

unique numerical cluster identifier Mandatory

**Response**

```json
HTTP Status 200
```

devices list of devices that have been changed, only existing if action is ”update”

**Sample**

```json
GET /json/structure/clusterRemoveDevice?deviceID=3504175fe000000000005854&groupID=16
{
ok:  true ,
result: {
action: update,
devices: [
{
id:  ”3504175fe000000000005854” ,
name:  ”” ,
functionID: 4144,
productRevision: 788,
productID: 1234,
hwInfo:  ”GE − TKM210” ,
meterDSID:  ”3504175fe0000010000004d9” ,
busID: 241,
zoneID: 1,
isPresent:  true ,
lastDiscovered:  ”2012 − 11 − 22￿10:35:05” ,
firstSeen:  ”2012 − 11 − 19￿14:34:02” ,
```

```json
inactiveSince:  ”1970 − 01 − 01￿01:00:00” ,
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
```
