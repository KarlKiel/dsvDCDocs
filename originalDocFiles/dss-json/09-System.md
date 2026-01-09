9 System
========

System

9.1
---

System Information

9.1.1
-----

Returns the version of the digitalSTROM Server software.

**Synopsis**

```json
HTTP GET /json/system/version
```

Parameter

None

**Response**

```json
HTTP Status 200
```

distroVersion the host platform firmware release (since v1.10)

**Sample**

```json
GET /json/system/version
{
ok:  true ,
result:
{
version:  ”dSS￿v1.31.1￿(git:2acc82fe90f273a788fb573b07419f29f369a02a)￿(oebuild@builder)” ,
distroVersion:  ”Angstrom￿2010.4 − devel − 20111031” ,
Hardware:  ”￿AIZO￿digitalSTROM￿Server” ,
Revision:  ”￿0000” ,
Serial:  ”￿0000000000000000” ,
EthernetID:  ”a8:99:5c:c0:00:27” ,
MachineID:  ”603a932537518b121da4ffad00000037” ,
Kernel:  ”Linux￿version￿2.6.32.8￿(jin@vsrv − pilot − feedback)￿(gcc￿version￿4.3.3￿(GCC)￿)￿#1￿Mon￿Jan￿31￿18:55:47￿CET￿2011”
}
}
```

9.1.2
-----

time

Gets the installation time.

**Synopsis**

```json
HTTP GET /json/system/time
```

Parameter

None

**Response**

```json
HTTP Status 200
```

timezone timezone description string

**Sample**

```json
GET /json/system/time
{
ok:  true ,
result:
{
time:  1448982580,
tzoffset: 3600,
daylight:  false ,
timezone:  ”Europe/Berlin”
}
}
```

9.1.3
-----

getDSID

Returns the dSUID and dSID of the digitalSTROM Server.

**Synopsis**

```json
HTTP GET /json/system/getDSID
```

Parameter

None

**Response**

```json
HTTP Status 200
```

dSUID dSUID of the dSS

**Sample**

```json
GET /json/systme/getDSID
{
ok:  true ,
result :
{
dSID:  ”303505d7f800182000c00027” ,
dSUID:  ”303505d7f80000000000182000c0002700”
}
}
```

9.2
---

Authentication

9.2.1
-----

login

Creates a new session using the provided credentials.

**Synopsis**

```json
HTTP GET /json/system/login
```

| Parameter | Description | Remarks |
|---|---|---|
| user | usernamestring | Mandatory |
| password | passwordstring | Mandatory |

user name string Mandatory

**Response**

```json
HTTP Status 200
```

result.token session token as string

**Sample**

```json
GET /json/system/login?user=dssadmin\&password=dssadmin
{
”ok”  :  true ,
”result”  : {  ”token”  :  ”cea026b6f9d69e57e030736076285da77dbf117d24dbec69e349b2fb4ab7425e”  }
}
```

9.2.2
-----

logout

Destroys the session and signs out the user.

**Synopsis**

```json
HTTP GET /json/system/logout
```

None

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/system/logout {  ”ok”  :  true  }
```

9.2.3
-----

loggedInUser

Returns the name of the currently logged in user.

**Synopsis**

```json
HTTP GET /json/system/loggedInUser
```

None

**Response**

```json
HTTP Status 200
```

result.name name of the currently logged in user

Note: if noone is currently logged in, the result will be empty, i.e. name will be missing.

**Sample**

```json
GET /json/system/loggedInUser
{  ”ok”  :  true ,  ”result”  : {  ”name”  :  ”dssadmin”  } }
```

9.2.4
-----

requestApplicationToken

Returns a token for paswordless login. The token will need to be approved by a user first, the caller

must not be logged in.

**Synopsis**

```json
HTTP GET /json/system/requestApplicationToken
```

applicationName name of the application that requests the token Mandatory

**Response**

```json
HTTP Status 200
```

result.applicationToken application token as string

**Sample**

```json
GET /json/system/requestApplicationToken?applicationName=Example
{
”ok”  :  true ,
”result”  :
{
”applicationToken”  :  ”4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a”
}
}
```

9.2.5
-----

enableToken

Enables an application token, caller must be logged in.

**Synopsis**

```json
HTTP GET /json/system/enableToken
```

applicationToken application token as string Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/system/enableToken?applicationToken=4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a
{  ”ok”  :  true  }
```

9.2.6
-----

revokeToken

Revokes an application token, caller must be logged in.

**Synopsis**

```json
HTTP GET /json/system/revokeToken
```

applicationToken application token as string Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/system/revokeToken?applicationToken=4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a
{  ”ok”  :  true  }
```

9.2.7
-----

loginApplication

Creates a new session using the registered application token.

**Synopsis**

```json
HTTP GET /json/system/loginApplication
```

| Parameter | Description | Remarks |
|---|---|---|
| applicationToken | applicationtokenasstring | Mandatory |

loginToken application token as string Mandatory

**Response**

```json
HTTP Status 200
```

result.token session token as string

**Sample**

```json
GET /json/system/loginApplication?loginToken=4fa07386c77d7f32260066c83b58aece5814698376bd03f0e3b5764e58f0ec1a
{
”ok”  :  true ,
”result”  : {  ”token”  :  ”a84bfd1219512078d5537a4a5cd1c78084e6c4d3f8b0ef2ae3a2c81dff638822”  }
}
```

9.3
---

Logging

9.3.1
-----

logChannelSeverities

Returns all log channel severities.

**Synopsis**

```json
HTTP GET /json/system/logChannelSeverities/
```

**Response**

```json
HTTP Status 200
```

entries object logChannelId -> logSeverity (fatal,warning,notice,info,debug)

**Sample**

```json
GET /json/system/logChannelSeverities/
{
”ok”  :  true ,
”result”  : {
”entries”  : {
”DSS” :  ”info” ,
”VdcToken” :  ”info” ,
”dsLocale” :  ”notice” ,
”System” :  ”debug”
}
}
}
```

9.3.2
-----

logChannelSeverities/[id]

Returns the log channel severity.

**Synopsis**

```json
HTTP GET /json/system/logChannelSeverities/[id]
```

**Response**

```json
HTTP Status 200
```

value logSeverity (fatal,warning,notice,info,debug)

**Sample**

```json
GET /json/system/logChannelSeverities/DSS
{
”ok”  :  true ,
```

```json
”result”  : {
”value” :  ”info”
}
}
```

9.3.3
-----

logChannelSeverities/[id]?method=put

Sets the log channel severity.

**Synopsis**

```json
HTTP GET /json/system/logChannelSeverities/[id]?method=put&value=[severity]
```

| Parameter | Description | Remarks |
|---|---|---|
| method | put | Mandatory |
| value | logseverity(fatal,warning,notice,info,debug) | Mandatory |

log severity (fatal,warning,notice,info,debug) Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/system/logChannelSeverities/DSS?method=put&value=debug
{  ”ok”  :  true }
```
