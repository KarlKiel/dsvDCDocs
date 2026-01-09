8 Metering
==========

8
Metering
8.1
Metering
8.1.1
getResolution
Returns a list of time-series metering data resolutions stored on the digitalSTROM-Server.
Synopsis
HTTP GET /json/metering/getResolutions
Parameter
None
Response
HTTP Status 200
Parameter
Description
ok
boolean result of the call
result.resolutions
a list of supported resolutions
result.resolutions[…].resolution step size in thei resolution in seconds
Sample
GET /json/metering/getResolutions
{
”ok”: true,
”result”: {
”resolutions”: [
{
”resolution”: 1
},
{
”resolution”: 60
},
{
”resolution”: 900
},
{
”resolution”: 86400
},
{
”resolution”: 604800
},
{
”resolution”: 2592000
}
]
}
}
8.1.2
getSeries
Returns a list of all metering series stored on the digitalSTROM-Server.
Three types of series are available:
energy An energy meter counter.
energyDelta The total energy consumed during the previous time slot.
consumption The average power used during the previous time slot.
145

Synopsis
HTTP GET /json/metering/getSeries
Parameter
None
Response
HTTP Status 200
Parameter
Description
ok
boolean result of the call
result.series
a list of available time series
result.series[…].dsid dSID of the digitalSTROM-Meter for this series
result.series[…].type the series type
Sample
GET /json/metering/getSeries
{
”ok”: true,
”result”: {
”series”: [
{
”dsid”: ”3504175fe00000100000053e”,
”type”: ”energy”
},
{
”dsid”: ”3504175fe0000010000006b4”,
”type”: ”energyDelta”
},
{
”dsid”: ”3504175fe0000010000008a5”,
”type”: ”consumption”
},
]
}
}
8.1.3
getValues
Returns a time series of metering values with the specified properties.
All times are integers that represent UNIX timestamps (seconds since 1970-01-01).
The (optional) window selection parameters can be used in different combinations. Only two of the
three options can be used together in a call. The following table details the available combinations:
startTime return all available values starting at startTime until now.
endTime return all available values from the oldest available until endTime.
valueCount return the valueCount newest values
startTime and valueCount return valueCount values starting from startTime.
endTime and valueCount return valueCount values ending at endTime
startTime and endTime return the values between startTime and endTime.
146

Synopsis
HTTP GET /json/metering/getValues
Parameter
Parameter Description
Remarks
dsuid
request the data for this digitalSTROM-Meter
Mandatory
type
series type (according to the getSeries call)
Mandatory
resolution
series resolution (the digitalSTROM-Server will adjust the resolution to
the closest multiple of the available resolutions according to the ge-
tResolutions call)
Mandatory
unit
(only relevant for types ”energy” and ”energyDelta”) unit of the returned
metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.
Optional
startTime
start time (UNIX timestamp)
Optional
endTime
ent time (UNIX timestamp)
Optional
valueCount number of values (UNIX timestamp)
Optional
Response
HTTP Status 200
Parameter
Description
ok
boolean result of the call
result.meterID
dSID of the digitalSTROM-Meter
result.type
same as Request
result.resolution actual resolution of the data, might differ from the requested resolution
if it was not available.
result.values
array of time-value pairs
Sample
GET /json/metering/getValues?dsuid=3504175fe000000000100000063a00&type=energy&resolution=60&unit=Ws&valueCount=5
{
”ok”: true,
”result”: {
”meterID”: ”3504175fe00000100000063a”,
”type”: ”energy”,
”unit”: ”Ws”,
”resolution”: ”60”,
”values”: [
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
147

1352906280,
47562600
]
]
}
}
8.1.4
getAggregatedValues
Returns the sum of time series of metering values with the specified properties for all digitalSTROM-
Meter’s.
Synopsis
HTTP GET /json/metering/getAggregatedValues
Parameter
Parameter Description
Remarks
dsuid
request the data as sum for a list of digitalSTROM-Meter, argument can
be a single dsuid, a list of dsuids given in ”.meters(dsuid1, dsuid2)” syn-
tax, or the meta command .meters(all)
Mandatory
type
series type (according to the getSeries call)
Mandatory
resolution
series resolution (the digitalSTROM-Server will adjust the resolution to
the closest multiple of the available resolutions according to the ge-
tResolutions call)
Mandatory
unit
(only relevant for types ”energy” and ”energyDelta”) unit of the returned
metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.
Optional
startTime
start time (UNIX timestamp)
Optional
endTime
ent time (UNIX timestamp)
Optional
valueCount number of values (UNIX timestamp)
Optional
Response
HTTP Status 200
Parameter
Description
ok
boolean result of the call
result.meterID
array of dSUID’s of the requested digitalSTROM-Meter’s
result.type
same as Request
result.resolution actual resolution of the data, might differ from the requested resolution
if it was not available.
result.values
array of summed up result values according to the given time period
Sample
GET /json/metering/getAggregatedValues?
dsuid=.meters(303505d7f8000000000002c00000379c00,3504175fe000000000000010000004d900)&
type=energy&resolution=60&unit=Ws&valueCount=5
{
”result”: {
”meterID”: [
148

”303505d7f8000000000002c00000379c00”,
”3504175fe000000000000010000004d900”
],
”type”: ”energy”,
”unit”: ”Ws”,
”resolution”: ”60”,
”values”: [
[ 
1448980080, 
9559790.0
], [ 
1448980140, 
9560116.0
], [ 
1448980200, 
9560438.0
], [ 
1448980260, 
9560760.0
]
]
},
”ok”: true
}
8.1.5
getLatest
Returns the latest available metering values.
Synopsis
HTTP GET /json/metering/getLatest
Parameter
Parameter Description
Remarks
from
the dSID of the requested digitalSTROM-Meters. It uses a Set-Syntax:
”.meters(dsid1,dsid2,…)” and ”.meters(all)”
Mandatory
type
series type (according to the getSeries call)
Mandatory
unit
(only relevant for types ”energy” and ”energyDelta”) unit of the returned
metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.
Optional
Response
HTTP Status 200
Parameter
Description
ok
boolean result of the call
result.values
array of results
result.values[…].dsid
dSID of the digitalSTROM-Meter
result.values[…].value the latest metering value
result.values[…].date
date and time when the latest metering value was recorded
Sample
GET /json/metering/getLatest?from=.meters(3504175fe00000100000063a,3504175fe0000010000008c4)&type=energy&unit=Ws
{
”ok”: true,
”result”: {
149

”values”: [
{
”dsid”: ”3504175fe00000100000063a”,
”value”: 49414887,
”date”: ”2012−11−19￿13:49:41”
},
{
”dsid”: ”3504175fe0000010000008c4”,
”value”: 151215631,
”date”: ”2012−11−19￿10:29:29”
}
]
}
}
8.1.6
getAggregatedLatest
Returns the sum of latest metering values for all digitalSTROM-Meter’s.
Synopsis
HTTP GET /json/metering/getAggregatedLatest
Parameter
Parameter Description
Remarks
from
the dSID of the requested digitalSTROM-Meters. It uses a Set-Syntax:
”.meters(dsid1,dsid2,…)” and ”.meters(all)”
Mandatory
type
series type (according to the getSeries call)
Mandatory
unit
(only relevant for types ”energy” and ”energyDelta”) unit of the returned
metering values. Options are ”Wh” and ”Ws”. Defaults to ”Wh”.
Optional
Response
HTTP Status 200
Parameter
Description
ok
boolean result of the call
result.values
array of results
result.values[…].dSUID dSUID of the digitalSTROM-Meter
result.values[…].dsid
dSID of the digitalSTROM-Meter (deprecated)
result.values[…].value
the summed up latest metering value
result.values[…].date
date and time when the latest metering value was recorded
Sample
GET /json/metering/getAggregatedLatest?
from=.meters(303505d7f8000000000002c00000379c00,3504175fe000000000000010000004d900)&
type=energy&unit=Ws
{
”result”: {
”values”: [
{
”dsid”: [
”303505d7f80002c00000379c”,
”3504175fe0000010000004d9”
],
”dSUID”: [
150

”303505d7f8000000000002c00000379c00”,
”3504175fe000000000000010000004d900”
],
”value”:  4214,
”date”: ”2015−12−01￿15:38:16”
}
]
},
”ok”: true
}
151

