5 Circuit
=========

Circuit

5.1
---

Common

Every /json/circuit/ function uses a common selection scheme for the connected infrastructure compo-

nent to which the command refers to. This component can bei either a connected digitalSTROM-Meter

or an IP Device Connector VDC.

The parameter ”dsuid” must be given to identify the component which is a string value of the dSUID.

The legacy parameter ”id” can be given to identify a dSM, where ”id” is a string value of the dSID.

| Parameter | Description | Remarks |
|---|---|---|
| dsuid | dSUIDNumberoftheinfrastructurecomponent | Mandatory |
| id | dSIDNumberofthedigitalSTROM-Meter | Legacyalternativetodsuid |

dSUID Number of the infrastructure component Mandatory

A missing dsuid identifier result in the following error message to be returned.

```json
{
”ok” :  false ,
”message” :  ”Missing￿parameter￿dsuid”
}
```

If a dSUID identifier does not match any actually known component in the installation the following error

message is returned.

```json
{
”ok” :  false ,
”message” :  ”Could￿not￿find￿dSMeter￿with￿given￿dsuid”
}
```

5.2
---

Name

5.2.1
-----

getName

Returns the user defined name of the zone.

**Synopsis**

```json
HTTP GET /json/circuit/getName
```

None

**Response**

```json
HTTP Status 200
```

name identifier string for the digitalSTROM-Meter

**Sample**

```json
GET /json/circuit/getName?id=3504175fe0000010000004d5
{
”ok” : true ,
”result”  : {
”name”  :  ”Wohnen/Flur/Eingang”
}
}
```

5.2.2
-----

setName

Sets the zone name.

**Synopsis**

```json
HTTP GET /json/circuit/setName
```

| Parameter | Description | Remarks |
|---|---|---|
| newName | identifierstringforthedigitalSTROM-Meter | Mandatory |

identifier string for the digitalSTROM-Meter Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/circuit/setName?id=3504175fe0000010000004d5&newName= ”Wohnen”
{
”ok” : true
}
```

5.3
---

Energy Meter

5.3.1
-----

getConsumption

Returns the current measurent of the power consumption on this circuit.

**Synopsis**

```json
HTTP GET /json/circuit/getConsumption
```

None

**Response**

```json
HTTP Status 200
```

consumption Current power consumption [W]

**Sample**

```json
GET /json/circuit/getConsumption?id=3504175fe0000010000004d5
{
”ok” :  true ,
”result”  : {
”consumption” : 725
}
}
```

5.3.2
-----

getEnergyMeterValue

Returns the current measurent of the power consumption on this circuit.

**Synopsis**

```json
HTTP GET /json/circuit/getEnergyMeterValue
```

None

**Response**

```json
HTTP Status 200
```

meterValue Energy Meter Value [Ws]

**Sample**

```json
GET /json/circuit/getEnergyMeterValue?id=3504175fe0000010000004d5
{
”ok” :  true ,
”result” : {
”meterValue” : 1438467
}
}
```

5.4
---

Configuration

5.4.1
-----

learnIn

Enable and allow to register new devices and establish new connections. Typically the registration of

new devices is a teach-in process that requires action e.g. button press on the physical device itself to

pair with a new peer.

**Synopsis**

```json
HTTP GET /json/circuit/learnIn
```

| meterValue | EnergyMeterValue[Ws] |
|---|---|

time in seconds until the lean in process is disabled again mandantory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/circuit/learnIn?dsuid=0963dd2d722d5dc6c0ecb2aa7465465600&timeout=30
{
”ok” : true ,
}
```

5.4.2
-----

learnOut

Revert the teach-in process and deregister devices.

**Synopsis**

```json
HTTP GET /json/circuit/learnOut
```

| Parameter | Description | Remarks |
|---|---|---|
| timeout | timeinsecondsuntiltheleanoutprocessisdisabledagain | mandantory |
| params | devicespecificparameters | optional,key/valuepairsencodedjsonobject |

time in seconds until the lean out process is disabled again mandantory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/circuit/learnOut?dsuid=0963dd2d722d5dc6c0ecb2aa7465465600&timeout=30
{
”ok” : true
}
```

5.4.3
-----

firmwareCheck

Test for firmware verification and availability of updates.

**Synopsis**

```json
HTTP GET /json/circuit/firmwareCheck
```

None

**Response**

```json
HTTP Status 200
```

status firmware status code: ”ok”, ”error”, ”update”

the firmware status could not be checked, e.g. due to missing connectivity

update an update is available and ready for installation

**Sample**

```json
GET /json/circuit/firmwareCheck?dsuid=0963dd2d722d5dc6c0ecb2aa7465465600
{
”ok” : true ,
”status” :  ”update”
}
```

5.4.4
-----

firmwareUpdate

Start the firmware upgrade process. This process is running autonomously. Typically the device will

restart and register again.

**Synopsis**

```json
HTTP GET /json/circuit/firmwareUpdate
```

clearsettings request to reset all data to factory defaults optional

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/circuit/firmwareUpdate?dsuid=0963dd2d722d5dc6c0ecb2aa7465465600&clearsettings= true
{
”ok” : true
}
```

5.4.5
-----

storeAccessToken

Receives authentication token and related data for a device.

**Synopsis**

```json
HTTP GET /json/circuit/storeAccessToken
```

| Parameter | Description | Remarks |
|---|---|---|
| clearsettings | requesttoresetalldatatofactorydefaults | optional |

authorization data string, content is device specific mandantory

authScope scope, device specific

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/circuit/storeAccessToken?dsuid=0963dd2d722d5dc6c0ecb2aa7465465600&authData={ ”access\_token” :
”42EC27D0AD0616334CB670C29211ABF1693C71666C6158960DD51DFFB4B18150” ,  ”expires” : 86400}&
authScope=user@domain.com,ReadData
{
”ok” : true
}
```

5.5
---

Reregister devices

5.5.1
-----

reregisterDevices

Reregisters devices on selected controller.

**Synopsis**

```json
HTTP GET /json/circuit/reregisterDevices
```

| Parameter | Description | Remarks |
|---|---|---|
| dsuid | dSUIDofthedevice | Mandatory |

dSUID of the device Mandatory

**Response**

```json
HTTP Status 200
```

ok true

**Sample**

```json
GET /json/circuit/reregisterDevices?dsuid=3504175fe000000000017ef3
{
”ok” : true ,
}
```
