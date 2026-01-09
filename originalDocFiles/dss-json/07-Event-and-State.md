7 Event and State
=================

7
Event and State
7.1
Raise Event
7.1.1
raise
Raises an event and appends it to the digitalSTROM-Server event queue. Details of the digitalSTROM-
Server event processing can be found in the system-interfaces document.
Notice System events should be treated as reserved and must not be raised by external applica-
tions. In this term system events are events which originate from the digitalSTROM system lower
layers.
Synopsis
HTTP GET /json/event/raise
Parameter
Parameter Description
Remarks
name
identifier string for event
Mandatory
parameter list of key=value pairs, seperated with semicolons Optional
Response
HTTP Status 200
ok true
Sample
GET /json/event/raise?name=highlevelevent&parameter=id=1026;value=0;index=1
{
”ok”:true
}
7.2
Event Subscription
7.2.1
subscribe
Subscribe to an event with the given name and registers the callers subscriptionId. A unique subscrip-
tionId can be selected by the subscriber. It is possible to subscribe to several events reusing the same
subscriptionId.
Synopsis
HTTP GET /json/event/subscribe
Parameter
Description
Remarks
name
identifier string for the event Mandatory
subscriptionID numerical unique value
Mandatory
Parameter
141

Response
HTTP Status 200
ok true
Sample
GET /json/event/subscribe?name=deviceSensorEvent&subscriptionID=42
{
”ok”:true
}
7.2.2
unsubscribe
Unsubscribes for the previously registered events by giving the event name and the unique subscrip-
tionId.
Synopsis
HTTP GET /json/event/unsubscribe
Parameter
Description
Remarks
name
identifier string for the event Mandatory
subscriptionID numerical unique value
Mandatory
Parameter
Response
HTTP Status 200
ok true
If there is no registered session for the given event name the following error message is returned.
{
ok: false
message: ”Event￿”callScene”￿is￿not￿subscribed￿in￿this￿session”
}
If the subscriptionId is unknown to the digitalSTROM-Server the following error message is returned.
{
ok: false
message: ”Token￿not￿found!”
}
Sample
GET /json/event/unsubscribe?name=callScene&subscriptionID=42
{
ok: true
}
142

7.2.3
get
Get event and context information for an event subscription. All events subscribed with the given Id will
be handled by this call. An optional timeout value in milliseconds can be specified and will block the
call until either an event or the timeout occurs. If the timeout value is zero or missing the call will not
timeout.
Synopsis
HTTP GET /json/event/get
Parameter
Description
Remarks
subscriptionID numerical unique value
Mandatory
timeout
numerical value, timeout in milli seconds Optional
Parameter
Response
HTTP Status 200
events array of events
Sample
GET /json/event/get?subscriptionID=42&timeout=60000
{
ok: true
result: {
events: [ ]
}
}
GET /json/event/get?subscriptionID=42&timeout=60000
{
ok: true
result: {
events: [
{
name: ”callScene”
properties: {
groupID: ”1”
sceneID: ”8”
zoneID: ”1241”
originDeviceID: ”3504175fe000000000005854”
}
}
]
}
}
7.3
State
7.3.1
set
Sets the value of a system state. Details of digitalSTROM-Server system states can be found in the
system-interfaces document.
Notice Only a subset of the system states can be changed by this method. Many systems states
reflect a physical status of an e.g. input line and cannot be modified.
143

Synopsis
HTTP GET /json/state/set
Parameter
Parameter Description
Remarks
name
identifier for the state
mandatory
value
new value
mandatory
addon
specify the owner of the state optional
Response
HTTP Status 200
ok true
Sample
GET /json/state/set?name=heating_system&value=off
{
”ok”:true
}
GET /json/state/set?addon=system−addon−user−defined−states&name=1484843926&value=active
{
”ok”:true
}
144

