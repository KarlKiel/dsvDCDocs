9 System
========

9
System
9.1
System Information
9.1.1
version
Returns the version of the digitalSTROM Server software.
Synopsis
HTTP GET /json/system/version
Parameter
None
Response
HTTP Status 200
version
the dSS application version
distroVersion the host platform firmware release (since v1.10)
Hardware
the host platform hardware identifier (since v1.10)
Revision
the host platform hardware revision number (since v1.10)
Serial
the host platform hardware serial number (since v1.10)
Ethernet
the host platform IEEE Mac address (since v1.10)
MachineID
the host system unique id (since v1.10)
Kernel
the host system Linux kernel release string (since v1.10)
Sample
GET /json/system/version
{
ok: true,
result:
{
version: ”dSS￿v1.31.1￿(git:2acc82fe90f273a788fb573b07419f29f369a02a)￿(oebuild@builder)”,
distroVersion: ”Angstrom￿2010.4−devel−20111031”,
Hardware: ”￿AIZO￿digitalSTROM￿Server”,
Revision: ”￿0000”,
Serial: ”￿0000000000000000”,
EthernetID: ”a8:99:5c:c0:00:27”,
MachineID: ”603a932537518b121da4ffad00000037”,
Kernel: ”Linux￿version￿2.6.32.8￿(jin@vsrv−pilot−feedback)￿(gcc￿version￿4.3.3￿(GCC)￿)￿#1￿Mon￿Jan￿31￿18:55:47￿CET￿2011”
}
}
9.1.2
time
Gets the installation time.
Synopsis
HTTP GET /json/system/time
Parameter
None
Response
HTTP Status 200
152

time
number of seconds since the Epoch, 1970-01-01 00:00:00 +0000 (UTC)
offset
offset in seconds east to GMT
daylight
boolean flag indicating if daylight saving is currently active
timezone timezone description string
Sample
GET /json/system/time
{
ok: true,
result:
{
time:  1448982580,
tzoffset: 3600,
daylight: false,
timezone: ”Europe/Berlin”
}
}
9.1.3
getDSID
Returns the dSUID and dSID of the digitalSTROM Server.
Synopsis
HTTP GET /json/system/getDSID
Parameter
None
Response
HTTP Status 200
dSID
dSID = SGTIN-96 of the dSS
dSUID dSUID of the dSS
Sample
GET /json/systme/getDSID
{
ok: true,
result :
{
dSID: ”303505d7f800182000c00027”,
dSUID: ”303505d7f80000000000182000c0002700”
}
}
9.2
Authentication
9.2.1
login
Creates a new session using the provided credentials.
Synopsis
HTTP GET /json/system/login
153

Parameter
Parameter Description
Remarks
user
user name string Mandatory
password
password string
Mandatory
Response
HTTP Status 200
result.token session token as string
Sample
GET /json/system/login?user=dssadmin\&password=dssadmin
{
”ok” : true,
”result” : { ”token” : ”cea026b6f9d69e57e030736076285da77dbf117d24dbec69e349b2fb4ab7425e” }
}
9.2.2
logout
Destroys the session and signs out the user.
Synopsis
HTTP GET /json/system/logout
Parameter
None
Response
HTTP Status 200
Sample
GET /json/system/logout { ”ok” : true }
9.2.3
loggedInUser
Returns the name of the currently logged in user.
Synopsis
HTTP GET /json/system/loggedInUser
Parameter
None
154

Response
HTTP Status 200
result.name name of the currently logged in user
Note: if noone is currently logged in, the result will be empty, i.e. name will be missing.
Sample
GET /json/system/loggedInUser
{ ”ok” : true, ”result” : { ”name” : ”dssadmin” } }
9.2.4
requestApplicationToken
Returns a token for paswordless login. The token will need to be approved by a user first, the caller
must not be logged in.
Synopsis
HTTP GET /json/system/requestApplicationToken
Parameter
Description
Remarks
applicationName name of the application that requests the token Mandatory
Parameter
Response
HTTP Status 200
result.applicationToken application token as string
Sample
GET /json/system/requestApplicationToken?applicationName=Example
{
”ok” : true,
”result” :
{
”applicationToken” : ”4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a”
}
}
9.2.5
enableToken
Enables an application token, caller must be logged in.
Synopsis
HTTP GET /json/system/enableToken
Parameter
155

Parameter
Description
Remarks
applicationToken application token as string Mandatory
Response
HTTP Status 200
Sample
GET /json/system/enableToken?applicationToken=4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a
{ ”ok” : true }
9.2.6
revokeToken
Revokes an application token, caller must be logged in.
Synopsis
HTTP GET /json/system/revokeToken
Parameter
Description
Remarks
applicationToken application token as string Mandatory
Parameter
Response
HTTP Status 200
Sample
GET /json/system/revokeToken?applicationToken=4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a
{ ”ok” : true }
9.2.7
loginApplication
Creates a new session using the registered application token.
Synopsis
HTTP GET /json/system/loginApplication
Parameter Description
Remarks
loginToken application token as string Mandatory
Response
HTTP Status 200
result.token session token as string
156

Sample
GET /json/system/loginApplication?loginToken=4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a
{
”ok” : true,
”result” : { ”token” : ”a84bfd1219512078d5537a4a5cd1c78084e6c4d3f8b0ef2ae3a2c81dff638822” }
}
9.3
Logging
9.3.1
logChannelSeverities
Returns all log channel severities.
Synopsis
HTTP GET /json/system/logChannelSeverities/
Response
HTTP Status 200
entries object logChannelId -> logSeverity (fatal,warning,notice,info,debug)
Sample
GET /json/system/logChannelSeverities/
{
”ok” : true,
”result” : {
”entries” : {
”DSS”: ”info”,
”VdcToken”: ”info”,
”dsLocale”: ”notice”,
”System”: ”debug”
}
}
}
9.3.2
logChannelSeverities/[id]
Returns the log channel severity.
Synopsis
HTTP GET /json/system/logChannelSeverities/[id]
Response
HTTP Status 200
value logSeverity (fatal,warning,notice,info,debug)
Sample
GET /json/system/logChannelSeverities/DSS
{
”ok” : true,
157

”result” : {
”value”: ”info”
}
}
9.3.3
logChannelSeverities/[id]?method=put
Sets the log channel severity.
Synopsis
HTTP GET /json/system/logChannelSeverities/[id]?method=put&value=[severity]
Parameter Description
Remarks
method
put
Mandatory
value
log severity (fatal,warning,notice,info,debug) Mandatory
Parameter
Response
HTTP Status 200
Sample
GET /json/system/logChannelSeverities/DSS?method=put&value=debug
{ ”ok” : true}
158

