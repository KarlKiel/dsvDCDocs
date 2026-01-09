8 Metering
==========

Metering

8.1
---

Metering

8.1.1
-----

getResolution

Returns a list of time-series metering data resolutions stored on the digitalSTROM-Server.

**Synopsis**

```json
HTTP GET /json/metering/getResolutions
```

None

**Response**

```json
HTTP Status 200
```

result.resolutions[…].resolution step size in thei resolution in seconds

**Sample**

```json
GET /json/metering/getResolutions
{
”ok” :  true ,
”result” : {
”resolutions” : [
{
”resolution” : 1
},
{
”resolution” : 60
},
{
”resolution” : 900
},
{
”resolution” : 86400
},
{
”resolution” : 604800
},
{
”resolution” : 2592000
}
}
}
```

8.1.2
-----

getSeries

Returns a list of all metering series stored on the digitalSTROM-Server.

Three types of series are available:

energy  An energy meter counter.

energyDelta  The total energy consumed during the previous time slot.

consumption  The average power used during the previous time slot.

**Synopsis**

```json
HTTP GET /json/metering/getSeries
```

None

**Response**

```json
HTTP Status 200
```

result.series[…].dsid dSID of the digitalSTROM-Meter for this series

result.series[…].type the series type

**Sample**

```json
GET /json/metering/getSeries
{
”ok” :  true ,
”result” : {
”series” : [
{
”dsid” :  ”3504175fe00000100000053e” ,
”type” :  ”energy”
},
{
”dsid” :  ”3504175fe0000010000006b4” ,
”type” :  ”energyDelta”
},
{
”dsid” :  ”3504175fe0000010000008a5” ,
”type” :  ”consumption”
},
}
}
```

8.1.3
-----

getValues

Returns a time series of metering values with the specified properties.

All times are integers that represent UNIX timestamps (seconds since 1970-01-01).

The (optional) window selection parameters can be used in different combinations. Only two of the

three options can be used together in a call. The following table details the available combinations:

startTime  return all available values starting at startTime until now.

endTime  return all available values from the oldest available until endTime.

valueCount  return the valueCount newest values

startTime and valueCount  return valueCount values starting from startTime.

endTime and valueCount  return valueCount values ending at endTime

startTime and endTime  return the values between startTime and endTime.

**Synopsis**

```json
HTTP GET /json/metering/getValues
```

| Parameter | Description | Remarks |
|---|---|---|
| dsuid | requestthedataforthisdigitalSTROM-Meter | Mandatory |
| type | seriestype(accordingtothegetSeriescall) | Mandatory |
| resolution | seriesresolution(thedigitalSTROM-Serverwilladjusttheresolutionto | Mandatory |
| unit | the closest multiple of the available resolutions according to the ge- | Optional |
| startTime | tResolutionscall) | Optional |
| endTime | (onlyrelevantfortypes”energy”and”energyDelta”)unitofthereturned | Optional |
| valueCount | meteringvalues. Optionsare”Wh”and”Ws”. Defaultsto”Wh”. | Optional |
|  | starttime(UNIXtimestamp) |  |
|  | enttime(UNIXtimestamp) |  |
|  | numberofvalues(UNIXtimestamp) |  |

the closest multiple of the available resolutions according to the ge-

metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.

valueCount number of values (UNIX timestamp)

**Response**

```json
HTTP Status 200
```

result.resolution actual resolution of the data, might differ from the requested resolution

**Sample**

```json
GET /json/metering/getValues?dsuid=3504175fe000000000100000063a00&type=energy&resolution=60&unit=Ws&valueCount=5
{
”ok” :  true ,
”result” : {
”meterID” :  ”3504175fe00000100000063a” ,
”type” :  ”energy” ,
”unit” :  ”Ws” ,
”resolution” :  ”60” ,
”values” : [
[
1352906040,
47562600
],
[
1352906100,
47562600
],
[
1352906160,
47562600
],
[
1352906220,
47562600
],
[
```

```json
1352906280,
47562600
]
]
}
}
```

8.1.4
-----

getAggregatedValues

Returns the sum of time series of metering values with the specified properties for all digitalSTROM-

Meter’s.

**Synopsis**

```json
HTTP GET /json/metering/getAggregatedValues
```

| Parameter | Description | Remarks |
|---|---|---|
| dsuid | requestthedataassumforalistofdigitalSTROM-Meter,argumentcan | Mandatory |
| type | beasingledsuid,alistofdsuidsgivenin”.meters(dsuid1,dsuid2)”syn- | Mandatory |
| resolution | tax,orthemetacommand.meters(all) | Mandatory |
| unit | seriestype(accordingtothegetSeriescall) | Optional |
| startTime | seriesresolution(thedigitalSTROM-Serverwilladjusttheresolutionto | Optional |
| endTime | the closest multiple of the available resolutions according to the ge- | Optional |
| valueCount | tResolutionscall) | Optional |
|  | (onlyrelevantfortypes”energy”and”energyDelta”)unitofthereturned |  |
|  | meteringvalues. Optionsare”Wh”and”Ws”. Defaultsto”Wh”. |  |
|  | starttime(UNIXtimestamp) |  |
|  | enttime(UNIXtimestamp) |  |
|  | numberofvalues(UNIXtimestamp) |  |

the closest multiple of the available resolutions according to the ge-

metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.

valueCount number of values (UNIX timestamp)

**Response**

```json
HTTP Status 200
```

result.resolution actual resolution of the data, might differ from the requested resolution

**Sample**

```json
GET /json/metering/getAggregatedValues?
dsuid=.meters(303505d7f8000000000002c00000379c00,3504175fe000000000000010000004d900)&
type=energy&resolution=60&unit=Ws&valueCount=5
{
”result” : {
”meterID” : [
```

```json
”303505d7f8000000000002c00000379c00” ,
”3504175fe000000000000010000004d900”
],
”type” :  ”energy” ,
”unit” :  ”Ws” ,
”resolution” :  ”60” ,
”values” : [
1448980080,
```

9559790.0
---------

```json
], [
1448980140,
```

9560116.0
---------

```json
], [
1448980200,
```

9560438.0
---------

```json
], [
1448980260,
```

9560760.0
---------

```json
},
”ok” :  true
}
```

8.1.5
-----

getLatest

Returns the latest available metering values.

**Synopsis**

```json
HTTP GET /json/metering/getLatest
```

| Parameter | Description | Remarks |
|---|---|---|
| from | the dSID of the requested digitalSTROM-Meters. It uses a Set-Syntax: | Mandatory |
| type | ”.meters(dsid1,dsid2,…)”and”.meters(all)” | Mandatory |
| unit | seriestype(accordingtothegetSeriescall) | Optional |
|  | (onlyrelevantfortypes”energy”and”energyDelta”)unitofthereturned |  |
|  | meteringvalues. Optionsare”Wh”and”Ws”. Defaultsto”Wh”. |  |

the dSID of the requested digitalSTROM-Meters. It uses a Set-Syntax:

metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.

**Response**

```json
HTTP Status 200
```

result.values[…].value the latest metering value

**Sample**

```json
GET /json/metering/getLatest?from=.meters(3504175fe00000100000063a,3504175fe0000010000008c4)&type=energy&unit=Ws
{
”ok” :  true ,
”result” : {
```

```json
”values” : [
{
”dsid” :  ”3504175fe00000100000063a” ,
”value” : 49414887,
”date” :  ”2012 − 11 − 19￿13:49:41”
},
{
”dsid” :  ”3504175fe0000010000008c4” ,
”value” : 151215631,
”date” :  ”2012 − 11 − 19￿10:29:29”
}
}
}
```

8.1.6
-----

getAggregatedLatest

Returns the sum of latest metering values for all digitalSTROM-Meter’s.

**Synopsis**

```json
HTTP GET /json/metering/getAggregatedLatest
```

| Parameter | Description | Remarks |
|---|---|---|
| from | the dSID of the requested digitalSTROM-Meters. It uses a Set-Syntax: | Mandatory |
| type | ”.meters(dsid1,dsid2,…)”and”.meters(all)” | Mandatory |
| unit | seriestype(accordingtothegetSeriescall) | Optional |
|  | (onlyrelevantfortypes”energy”and”energyDelta”)unitofthereturned |  |
|  | meteringvalues. Optionsare”Wh”and”Ws”. Defaultsto”Wh”. |  |

the dSID of the requested digitalSTROM-Meters. It uses a Set-Syntax:

metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.

**Response**

```json
HTTP Status 200
```

result.values[…].dSUID dSUID of the digitalSTROM-Meter

**Sample**

```json
GET /json/metering/getAggregatedLatest?
from=.meters(303505d7f8000000000002c00000379c00,3504175fe000000000000010000004d900)&
type=energy&unit=Ws
{
”result” : {
”values” : [
{
”dsid” : [
”303505d7f80002c00000379c” ,
”3504175fe0000010000004d9”
],
”dSUID” : [
```

```json
”303505d7f8000000000002c00000379c00” ,
”3504175fe000000000000010000004d900”
],
”value” :  4214,
”date” :  ”2015 − 12 − 01￿15:38:16”
}
]
},
”ok” :  true
}
```
