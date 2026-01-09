3 Zone
======

3.1
---

Common

Every /json/zone/ function uses a common selection scheme for the zone to which the command refers

to. Either the parameter ”id” or ”name” must be given to identify the zone. The special value zero for

the ”id” maybe used to send the command as broadcast to all zones.

| Parameter | Description | Remarks |
|---|---|---|
| id | ZoneNumber | Optional |
| name | ZoneName | Optional |

Zone Number Optional

A missing zone identifier result in the following error message to be returned.

```json
{
”ok” :  false ,
”message” :  ”Need￿parameter￿name￿or￿id￿to￿identify￿zone”
}
```

If a zone identifier does not match any actually known zone in the installation the following error mes-

sage is returned.

```json
{
”ok” :  false ,
”message” :  ”Could￿not￿find￿zone￿with￿id￿’1250’”
}
```

3.2
---

Action

3.2.1
-----

getActions

Get the actions for specified  application .

**Synopsis**

```json
HTTP GET /json/zone/getActions
```

| Parameter | Description | Remarks |
|---|---|---|
| application | Numberofthetargetapplication | Mandatory |
| lang | LocaleCodebasedonLanguage_Regionpattern,e.g. en_EN,de_DE | Optional |

application Number of the target application

Locale Code based on Language_Region pattern, e.g. en_EN, de_DE Optional

**Response**

```json
HTTP Status 200
```

result.actions Array of actions with  action.id  and  action.title

If the  lang  parameter is omitted the  action.title  is translated to english as default.

**Sample**

```json
GET /json/zone/getActions?id=0&application=2
{
”result” :{
”actions” :[
{
”id” : ”app.moveUp” ,
”title” : ”Blinds￿Move￿Up”
},
{
”id” : ”app.moveDown” ,
”title” : ”Blinds￿Move￿Down”
},
{
”id” : ”app.stepUp” ,
”title” : ”Blinds￿Step￿Up”
},
{
”id” : ”app.stepDown” ,
”title” : ”Blinds￿Step￿Down”
},
{
”id” : ”app.sunProtection” ,
”title” : ”Blinds￿Sun￿Protection”
},
{
”id” : ”app.stop” ,
”title” : ”Blinds￿Stop”
}
]
},
”ok” : true
}
```

**Sample**

Example with  lang  parameter

```json
GET /json/zone/getActions?id=0&application=2&lang=de_DE
{
”result” :{
”actions” :[
{
”id” : ”app.moveUp” ,
”title” : ”Rollläden￿Hochfahren”
},
{
”id” : ”app.moveDown” ,
”title” : ”Rollläden￿Herunterfahren”
},
{
”id” : ”app.stepUp” ,
”title” : ”Rollläden￿1￿Schritt￿Hochfahren”
},
{
”id” : ”app.stepDown” ,
”title” : ”Rollläden￿1￿Schritt￿Runterfahren”
},
{
”id” : ”app.sunProtection” ,
”title” : ”Sonnenschutz￿Für￿Rollläden”
},
{
”id” : ”app.stop” ,
”title” : ”Rollläden￿Stoppen”
}
]
},
”ok” : true
}
```

3.2.2
-----

callAction

Execute the  action  in a zone or in a cluster for devices of specified application.

**Synopsis**

```json
HTTP GET /json/zone/callAction
```

| Parameter | Description | Remarks |
|---|---|---|
| action | Nameoftheactiontobeexecuted | Mandatory |
| application | Numberofthetargetapplication | Conditional |
| cluster | Nameofthetargetcluster | Optional |

Name of the action to be executed Mandatory

application Number of the target application

The  application  parameter can be omitted only if  cluster  exists. To call action within apartment, param

id  must be equal to  0 .

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/callAction?id=0&action=app.moveUp&application=2
{
”ok” : true
}
```

Example with the  cluster  parameter.

```json
GET /json/zone/callAction?id=0&action=app.moveUp&cluster=17
{
”ok” : true
}
```

List of all supported actions

Application Name Application

app.sunProtection Blinds Sun Protection Blinds

3.3
---

3.3.1
-----

getName

Returns the user defined name of the zone.

**Synopsis**

```json
HTTP GET /json/zone/getName
```

None

**Response**

```json
HTTP Status 200
```

name identifier string for the zone

**Sample**

```json
GET /json/zone/getName?id=1237
{
”ok” : true ,
”result”  :
{
”name”  :  ”Wohnen”
}
}
```

3.3.2
-----

setName

Sets the zone name.

**Synopsis**

```json
HTTP GET /json/zone/setName
```

| name | identifierstringforthezone |
|---|---|

identifier string for the zone Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setName?id=1237&newName= ”Wohnen”
{
”ok” : true
}
```

3.4
---

Scene

3.4.1
-----

callScene

Excutes the scene  sceneNumber  in a zone for a group of devices.

**Synopsis**

```json
HTTP GET /json/zone/callScene
```

sceneNumber Numerical value

Boolean value, if set a forced scene call is issued Optional

If the group parameters are omitted the command is sent as broadcast to all devices in a zone.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/callScene?id=1237&groupID=1&sceneNumber=5&force= true
{
”ok” : true
}
```

3.4.2
-----

saveScene

Tells devices to store their current output values as a default for the scene  sceneNumber .

**Synopsis**

```json
HTTP GET /json/zone/saveScene
```

sceneNumber Numerical value

Number of the target group Optional

If the group parameters are omitted the command is sent as broadcast to all devices in a zone.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/saveScene?id=1237&groupID=2&sceneNumber=17
{
”ok” : true
}
```

3.4.3
-----

undoScene

Tells devices to restore their output values to the previous state if the current scene matches the  sce-

neNumber .

**Synopsis**

```json
HTTP GET /json/zone/undoScene
```

sceneNumber Numerical value

Number of the target group Optional

If the group parameters are omitted the command is sent as broadcast to all devices in the zone.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/undoScene?id=1237&sceneNumber=65
{
”ok” : true
}
```

3.4.4
-----

sceneGetName

Get the user defined name for a scene  sceneNumber  within a group of a zone.

**Synopsis**

```json
HTTP GET /json/zone/sceneGetName
```

sceneNumber Numerical value

Number of the target group M/O

Either groupID or groupName must be supplied to this request.

**Response**

```json
HTTP Status 200
```

result.name the user defined name of the scene

**Sample**

```json
GET /json/zone/sceneGetName?id=1237&sceneNumber=19&groupID=1
{
”ok” : true
result ”:￿{
￿￿￿￿￿￿￿” name ”:￿” Fernsehen ”
￿￿￿}
}
```

3.4.5
-----

sceneSetName

Sets a user defined name for a scene  sceneNumber  within a group of a zone. This name is stored on

the digitalSTROM-Server only.

**Synopsis**

```json
HTTP GET /json/zone/sceneSetName
```

User defined name of the scene Mandatory

sceneNumber Numerical value

Either groupID or groupName must be supplied to this request.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/sceneSetName?id=1237&sceneNumber=17&groupID=2&newName= ”Fernsehen”
{
”ok” : true
}
```

3.4.6
-----

getReachableScenes

Returns a list of groups which can be controlled by pushbuttons which are actually present in the zone.

**Synopsis**

```json
HTTP GET /json/zone/getReachableScenes
```

Number of the target group Mandatory

groupName Name of the target group

Either groupID or groupName are required.

**Response**

```json
HTTP Status 200
```

result.reachableScens

array of scene numbers

result.userSceneNames array of user defined scene names

**Sample**

```json
GET /json/zone/getReachableScenes?id=1237&groupID=1
{
”ok” :  true ,
”result” : {
”reachableScenes” : [
0,
1,
5,
6,
17,
18,
19,
29,
30,
31,
38,
]
”userSceneNames” : [
{
”sceneNr” : 17,
”sceneName” :  ”Watch￿TV”
},
{
”sceneNr” : 18,
”sceneName” :  ”Reading”
}
]
}
}
```

3.4.7
-----

getLastCalledScene

Returns the  sceneNumber  which has been executed last for a group in a zone.

**Synopsis**

```json
HTTP GET /json/zone/getLastCalledScene
```

Number of the target group Optional

groupName Name of the target group

**Response**

```json
HTTP Status 200
```

result.scene the number of the last called scene

**Sample**

```json
GET /json/zone/getLastCalledScene?id=1237&groupID=1
{
”ok” :  true ,
”result” : {
”scene” : 17
}
}
GET /json/zone/getLastCalledScene?id=0
{
”ok” :  true ,
”result” : {
”scene” : 69
}
}
```

3.5
---

Value

3.5.1
-----

Set Output Value

Set the output value of a group of devices in a zone to a given value.

Notice  Setting output values directly bypasses the group state machine and is not recommended.

**Synopsis**

```json
HTTP GET /json/zone/setValue
```

Number of the target group Optional

groupName Name of the target group

If the group parameters are omitted the command is sent as broadcast to all devices in the selected

zone.

Notice  Setting output values without a group identification is strongly unrecommended.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setValue?id=1237&value=0&groupID=2
{
”ok” : true ,
}
```

3.5.2
-----

Blink

Executes the ”blink” function on a group of devices in a zone for identification purposes.

**Synopsis**

```json
HTTP GET /json/zone/blink
```

Number of the target group Optional

groupName Name of the target group

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/blink?id=1237&groupID=1
{
”ok” : true ,
}
```

3.5.3
-----

Send Sensor Value

Send a sensor value to a group of devices in a zone.

**Synopsis**

```json
HTTP GET /json/zone/pushSensorValue
```

sourceDSUID DSUID of the originating device Optional

If the group parameter is omitted the command is sent as broadcast to all devices in the selected zone.

The reference for the sensor type definitions can be found in the ds-basics document.

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/pushSensorValue?id=1237&sensorType=51&sensorValue=100&groupID=48
{
”ok” : true ,
}
```

3.5.4
-----

Set Status Field

Set the value of a group application status.

**Synopsis**

```json
HTTP GET /json/zone/setStatusField
```

The following fields and attributes are defined:

malfunction Apartment Ventilation (groupID 64) Indicates malfunction of the whole service or a device

Apartment Ventilation (groupID 64) Indicates service request for a device

The group status flags are available as status object in the property tree:

/json/property/query?query=/usr/states/zone.0.group.64.status.service(*)

The malfunction and service attributes can also be set by hardware sensor inputs.

| Parameter | Description | Remarks |
|---|---|---|
| groupID | Numberofthetargetgroup | Default”0”,optional |
| sourceDSUID | DSUIDoftheoriginatingdevice | Optional |
| sensorValue | Numericalvalue | Mandatory |
| sensorType | Numericaltypeofthesensor | Mandantory |

Number of the target group Default ”0”, optional

String value of the attribute Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setStatusField?id=0&groupID=64&field=malfunction&value=active
{
”ok” : true ,
}
```

3.6
---

Sensors

3.6.1
-----

Set Sensor Source

Set the source of a sensor in a zone to a given device source address. For example one might have

multiple temperature and humidity sensors in a a room and using this method he can select which one

to use for room temperature control.

**Synopsis**

```json
HTTP GET /json/zone/setSensorSource
```

sensorType Numerical value of the sensor type Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setSensorSource?id=1237&sensorType=9&dsuid=3504175fe00000000000000000016be700
{
”ok” :  true ,
}
```

3.6.2
-----

Clear Sensor Source

Remove all assignments for a particular sensor type in a zone.

**Synopsis**

```json
HTTP GET /json/zone/clearSensorSource
```

sensorType Numerical value of the sensor type Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/clearSensorSource?id=1237&sensorType=11
{
”ok” :  true ,
}
```

3.6.3
-----

Get Assigned Sensors

Returns the list of assigned sensor devices in a zone.

**Synopsis**

```json
HTTP GET /json/zone/getAssignedSensors
```

None

**Response**

```json
HTTP Status 200
```

sensorType Numerical value of the sensor type

**Sample**

```json
GET /json/zone/getAssignedSensors?id=1237
{
”ok” : true ,
”result” : {
”sensors” : [
{
”sensorType” : 9,
”dsuid” : 3504175fe00000000000000000016be700
},
{
”sensorType” : 11,
”dsuid” : 3504175fe00000000000000000016be700
}
]
}
}
```

3.6.4
-----

Get Sensor Values

Returns a list of sensor measurements relevant for a zone. The temperature, humidity, CO2 concen-

tration and brightness are sensor types that are tracked for a zone. Typically there is one device as a

zone reference for these values.

If there is no standard device defined for a sensor type or if no measurement is available there is neither

the value or time field returned.

**Synopsis**

```json
HTTP GET /json/zone/getSensorValues
```

Parameter

None

**Response**

```json
HTTP Status 200
```

CO2ConcentrationValueTime Timestamp of the CO2 concentration measurement

**Sample**

```json
GET /json/zone/getSensorValues?id=1237
{
”ok” :  true ,
”result” : {
”id” : 14236,
”name” :  ”Heizungsraum” ,
”values” : [
{
”TemperatureValue” : 26.5,
”TemperatureValueTime” :  ”2014 − 10 − 13T17:57:21.445+0200”
}
]
}
}
```

3.7
---

Heating

3.7.1
-----

Get Temperature Control Status

Get the current status of the zone temperature control.

**Synopsis**

```json
HTTP GET /json/zone/getTemperatureControlStatus
```

Parameter

None

**Response**

```json
HTTP Status 200
```

Control mode: 0=off; 1=pid-control; 2=zone-follower;

TemperatureValueTime Timestamp of last temperature data update, seconds

Timestamp of last set point change, seconds since

**Sample**

```json
GET /json/zone/getTemperatureControlStatus?id=1237
{
”ok” :  true ,
”result” :
{
”ControlMode” : 1,
”OperationMode” : 4,
”TemperatureValue” : 20.7,
”NominalValue” : 20.0,
”ControlValue” : 92.5,
”TemperatureValueTime” : 2014 − 10 − 08T18:21:05Z,
”NominalValueTime” : 2014 − 10 − 08T18:00:00Z,
”ControlValueTime” : 2014 − 10 − 08T18:22:00Z
}
}
```

3.7.2
-----

Get Temperature Control Configuration

Get the configuration of the zone temperature control.

**Synopsis**

```json
HTTP GET /json/zone/getTemperatureControlConfig
```

Parameter

None

**Response**

```json
HTTP Status 200
```

Control mode: 0=off; 1=pid-control; 2=zone-follower;

EmergencyValue Fixed control value in case of malfunction (mode 1

Control integrator anti wind up: 0=inactive, 1=active

**Sample**

```json
GET /json/zone/getTemperatureControlConfig?id=1237
{
”ok” :  true ,
”result” :
{
”ControlMode” : 1,
”EmergencyValue” : 50,
”CtrlKp” : 5.2,
”CtrlTs” : 240,
”CtrlTi” : 1,
”CtrlKd” : 1,
”CtrlImin” : 600,
”CtrlImax” : 2400,
”CtrlYmin” : 0,
”CtrlYmax” : 100,
”CtrlAntiWindUp” : 1
}
}
GET /json/zone/getTemperatureControlConfig?id=1237
{
”ok” :  true ,
”result” :
{
”ControlMode” : 2,
”ReferenceZone” : 0,
”CtrlOffset” : 10
}
}
GET /json/zone/getTemperatureControlConfig?id=1237
{
”ok” :  true ,
”result” :
{
”ControlMode” : 3
}
}
```

```json
GET /json/zone/getTemperatureControlConfig?id=1237
{
”ok” :  true ,
”result” :
{
”ManualValue” : 87.5,
”ControlMode” : 4
}
}
```

3.7.3
-----

Set Temperature Control Configuration

Set the configuration of the zone temperature control.

**Synopsis**

```json
HTTP GET /json/zone/setTemperatureControlConfig
```

Parameter

Optional for Con-

Optional for Con-

EmergencyValue Fixed control value in case of malfunction Optional for Con-

Optional for Con-

Optional for Con-

Optional for Con-

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setTemperatureControlConfig?id=1237&ControlMode=2&ReferenceZone=4327&CtrlOffset= − 20
{
”ok” :  true ,
}
```

3.7.4
-----

Get Temperature Control Values

Get the temperature control operation mode preset values for a zone. Every control operation mode

has up to 15 presets defined.

**Synopsis**

```json
HTTP GET /json/zone/getTemperatureControlValues
```

Parameter

None

**Response**

```json
HTTP Status 200
```

Preset value for operation mode 0: ”Off”

Preset value for operation mode 1: ”Comfort”

Preset value for operation mode 2: ”Economy”

Preset value for operation mode 3: ”Not Used”

Preset value for operation mode 4: ”Night”

Preset value for operation mode 5: ”Holiday”

Preset value for operation mode 6: ”Cooling”

CoolingOff Preset value for operation mode 7: ”CoolingOff”

**Sample**

```json
GET /json/zone/getTemperatureControlValues?id=1237
{
”ok” :  true ,
”result” :
{
”Off” : 22.5,
”Comfort” : 21,
”Economy” : 20,
”NotUsed” : 18,
”Night” : 16,
”Holiday” : 4,
”Cooling” : 23,
”CoolingOff” : 50,
}
}
```

3.7.5
-----

Set Temperature Control Values

Set the temperature control operation mode preset values for a zone. Single values can be given and

others that do not change may be omitted.

Notice  For operation mode ”PID Control” the given values are nominal temperatures, and for op-

eration mode ”Fixed Values” the given values are absolute control values.

**Synopsis**

```json
HTTP GET /json/zone/setTemperatureControlValues
```

Parameter

Preset value for operation mode 0: ”Off”

Preset value for operation mode 1: ”Comfort”

Preset value for operation mode 2: ”Economy”

Preset value for operation mode 3: ”Not Used”

Preset value for operation mode 4: ”Night”

Preset value for operation mode 5: ”Holiday”

Preset value for operation mode 6: ”Cooling”

CoolingOff Preset value for operation mode 7: ”CoolingOff” Optional

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setTemperatureControlValues?id=1237&Comfort=22.5&Night=21
{
”ok” :  true
}
```

3.7.6
-----

Get Temperature Control Configuration v2

Get the temperature control configuration parameters.

**Synopsis**

```json
HTTP GET /json/zone/getTemperatureControlConfig2
```

Parameter

None

**Response**

```json
HTTP Status 200
```

Current Control Mode of the zone: off”, ”control”, ”zoneFollower”, ”fixed”, ”manual”

targetTemperatures Set point temperatures for each operation mode of the zone

emergencyValue Fixed control value in case of malfunction (int)

referenceZone Zone number of the reference zone (int)

controlValue Control value for manual mode (int)

**Sample**

```json
GET /json/zone/getTemperatureControlConfig2?id=1237
{
”mode”  :  ”control” ,
”targetTemperatures”  : {
”0” : 6,  ”1” : 23.5,  ”2” : 22,  ”3” : 19,
”4” : 18,  ”5” : 18,  ”6” : 22,  ”7” : 50,
”8” : 24,  ”9” : 28,  ”10” : 32,  ”11” : 30
},
”fixedValues”  : {
”0” : 0,  ”1” : 100,  ”2” : 90,  ”3” : 80,
”4” : 70,  ”5” : 25,  ”6” : 100,  ”7” : 0,
”8” : 80,  ”9” : 60,  ”10” : 40,  ”11” : 25
},
”controlMode”  : {
”emergencyValue” : 50,
”ctrlKp” : 5.2,
”ctrlTs” : 240,
”ctrlTi” : 1,
”ctrlKd” : 1,
”ctrlImin” : 600,
”ctrlImax” : 2400,
”ctrlYmin” : 0,
”ctrlYmax” : 100,
```

```json
”ctrlAntiWindUp” :  true
},
”zoneFollowerMode”  : {
”referenceZone” : 38523,
”ctrlOffset” : 10
},
”manualMode”  : {
”controlValue”  : 30
}
}
```

3.7.7
-----

Set Temperature Control Configuration v2

Set the configuration of the zone temperature control. The object content expected matches the ones

received by “getTemperatureControlConfig2”.

**Synopsis**

```json
HTTP GET /json/zone/setTemperatureControlConfig2
```

Parameter

Object with a collection of all fields that

Current Control Mode of the zone: ”off”,

”control”, ”zoneFollower”, ”fixed”, ”man-

targetTemperatures Object with set point temperatures for

Object with the zone follower related pa-

Object with the manual mode parameter

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/zone/setTemperatureControlConfig2?id=1237&mode=manual
{
”ok” :  true ,
}
GET /json/zone/setTemperatureControlConfig2?id=1237&targetTemperatures={ ”1” : 23.5,  ”2” : 22,  ”3” : 19,  ”8”  : 35}
{
”ok” :  true ,
}
GET /json/zone/setTemperatureControlConfig2?id=1237&fields={ ”mode” : ”control” ,  ”targetTemperatures” :{ ”1” : 23.5,  ”2” : 22,  ”3” : 19,  ”8”  :
35},  ”controlMode”  : { ”emergencyValue”  : 42}}
```

```json
{
”ok” :  true ,
}
GET /json/zone/setTemperatureControlConfig2?id=1237&fields={ ”mode” : ”manual” ,  ”manualMode” :{ ”contolValue”  : 45}}
{
”ok” :  true ,
}
```

3.7.8
-----

Set Temperature Control State

Modify the internal state of the temperature control for a zone.

**Synopsis**

```json
HTTP GET /json/zone/setTemperatureControlState
```

Notice  Obsolete and has been removed in dSS Release 1.42.

3.7.9
-----

Get Temperature Control Internals

Returns status information of the temperature control of a zone. Every controller attached to this re-

ports its internal configuration and algorithm status data.

**Synopsis**

```json
HTTP GET /json/zone/getTemperatureControlInternals
```

Parameter

None

**Response**

```json
HTTP Status 200
```

result.DSUID Object with the internal control parameters of this

Control mode: 0=off; 1=pid-control; 2=zone-follower;

Control state: 0=internal; 1=external; 2=exbackup;

CtrlTReference Control temperature

CtrlAntiWindUp Currently the anti wind up condition is active

**Sample**

```json
GET /json/zone/getTemperatureControlInternals?id=1237
{
”ok” : true ,
”result” :
{
”3504175fe0000000001000000006239100” : {
”ControlMode” : 1,
”ControlState” : 0,
”CtrlTRecent” : 20.50,
”CtrlTReference” : 21.00,
”CtrlTError” : 0.55,
”CtrlTErrorPrev” : 0.50,
”CtrlIntegral” : 82,
”CtrlYp” : 3.55,
”CtrlYi” : 23.1,
”CtrlYd” : 0,
”CtrlY” : 27,
”CtrlAntiWindUp” : 0
},
....
”3504175fe000000000100000000714a300” : {
”ControlMode” : 1,
”ControlState” : 2,
”CtrlTRecent” : 20.50,
”CtrlTReference” : 21.00,
”CtrlTError” : 0.55,
”CtrlTErrorPrev” : 0.50,
”CtrlIntegral” : 130,
”CtrlYp” : 3.55,
”CtrlYi” : 27.6,
”CtrlYd” : 0,
”CtrlY” : 29,
”CtrlAntiWindUp” : 0
},
}
}
```
