10 Property Tree
================

Property Tree

10.1
----

Basic Property Tree Operations

10.1.1
------

getString

Returns the string value of the property, this call will fail if the property is not of type ’string’.

**Synopsis**

```json
HTTP GET /json/property/getString
```

| Parameter | Description | Remarks |
|---|---|---|
| path | pathoftheproperty | Mandatory |

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

result.value string value of the property

**Sample**

```json
GET /json/property/getString?path=/system/version/version
{  ”ok”  :  true ,  ”result”  : {  ”value”  :  ”1.17.3”  } }
```

10.1.2
------

setString

Sets the string value of the property, this call will fail if the property is not of type ’string’.

**Synopsis**

```json
HTTP GET /json/property/setString
```

| result.value | stringvalueoftheproperty |
|---|---|

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/property/setString?path=/testpath/teststring\&value=testvalue
{  ”ok”  :  true  }
```

10.1.3
------

getInteger

Returns the integer value of the property, this call will fail if the property is not of type ’integer’.

**Synopsis**

```json
HTTP GET /json/property/getInteger
```

| Parameter | Description | Remarks |
|---|---|---|
| path | pathoftheproperty | Mandatory |

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

result.value integer value of the property

**Sample**

```json
GET /json/property/getInteger?path=/system/uptime
{  ”ok”  :  true ,  ”result”  : {  ”value”  : 7539 } }
```

10.1.4
------

setInteger

Sets the integer value of the property, this call will fail if the property is not of type ’integer’.

**Synopsis**

```json
HTTP GET /json/property/setInteger
```

| result.value | integervalueoftheproperty |
|---|---|

integer value of the property Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/property/setInteger?path=/testpath/testint\&value=1
{  ”ok”  :  true  }
```

10.1.5
------

getBoolean

Returns the boolean value of the property, this call will fail if the property is not of type ’boolean’.

**Synopsis**

```json
HTTP GET /json/property/getBoolean
```

| Parameter | Description | Remarks |
|---|---|---|
| path | pathoftheproperty | Mandatory |

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

result.value boolean value of the property

**Sample**

```json
GET /json/property/getBoolean?path=/config/subsystems/Metering/enabled
{  ”ok”  :  true ,  ”result”  : {  ”value”  :  true  } }
```

10.1.6
------

setBoolean

Returns the boolean value of the property, this call will fail if the property is not of type ’boolean’.

**Synopsis**

```json
HTTP GET /json/property/setBoolean
```

| result.value | booleanvalueoftheproperty |
|---|---|

boolean value of the property Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/property/setBoolean?path=/testpath/testbool\&value= true
{  ”ok”  :  true  }
```

10.1.7
------

getChildren

Returns an array of child nodes.

**Synopsis**

```json
HTTP GET /json/property/getChildren
```

| Parameter | Description | Remarks |
|---|---|---|
| path | pathofthenode | Mandatory |

path of the node Mandatory

**Response**

```json
HTTP Status 200
```

result[] result is an array of child nodes

**Sample**

```json
GET /json/property/getChildren?path=/system/host/interfaces/lo
{
”ok”  :  true ,
”result” :
{  ”name”  :  ”mac” ,  ”type”  :  ”string” },
{  ”name”  :  ”ip” ,  ”type”  :  ”string” },
{  ”name”  :  ”netmask” ,  ”type”  :  ”string” }
}
```

10.1.8
------

getType

Returns the type of the property, this can be “none”, “string”, “integer” or “boolean”.

**Synopsis**

```json
HTTP GET /json/property/getType
```

| result[] | resultisanarrayofchildnodes |
|---|---|

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

result.type type of the property

**Sample**

```json
GET /json/property/getType?path=/system/host/interfaces/lo/mac
{  ”ok”  :  true ,  ”result”  : {  ”type”  :  ”string”  } }
```

10.1.9
------

getFlags

Returns the flag values of a property.

**Synopsis**

```json
HTTP GET /json/property/getFlags
```

| Parameter | Description | Remarks |
|---|---|---|
| path | pathoftheproperty | Mandatory |

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

result.WRITEABLE information about the WRITEABLE flag

**Sample**

```json
GET /json/property/getFlags?path=/system/host/interfaces/lo/mac
{  ”ok”  :  true ,  ”result”  : {  ”READABLE”  :  true ,  ”WRITEABLE”  :  true ,  ”ARCHIVE”  :  false  } }
```

10.1.10
-------

setFlag

Sets a given flag of a property.

**Synopsis**

```json
HTTP GET /json/property/setFlag
```

| result.READABLE | informationabouttheREADABLEflag |
|---|---|
| result.WRITEABLE | informationabouttheWRITEABLEflag |
| result.ARCHIVE | informationabouttheARCHIVEflag |

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/property/setFlag?path=/system/host/interfaces/lo/mac\&flag=WRITEABLE\&value= true
{  ”ok”  :  true  }
```

10.1.11
-------

remove

Removes a property node.

**Synopsis**

```json
HTTP GET /json/property/remove
```

| Parameter | Description | Remarks |
|---|---|---|
| path | pathoftheproperty | Mandatory |

path of the property Mandatory

**Response**

```json
HTTP Status 200
```

**Sample**

```json
GET /json/property/remove?path=/testpath
{  ”ok”  :  true  }
```

10.2
----

Property Query

10.2.1
------

Returns a part of the tree specified by query. All queries start from the root. The properties to be

included have to be put in parentheses. A query to get all device from zone4 would look like this: ’/a-

partment/zones/zone4/*(ZoneID,name)’. More complex combinations (see example below) are also

possible.

**Synopsis**

```json
HTTP GET /json/property/query
```

| Parameter | Description | Remarks |
|---|---|---|
| query | querystring | Mandatory |

query string Mandatory

**Response**

```json
HTTP Status 200
```

result.value string value of the property

**Sample**

```json
GET /json/property/query?query=/apartment/zones/{*}(ZoneID,scenes)/groups/{*}(group,name)/scenes/{*}(scene,name)
{
”ok” : true ,
”result” :
{
”zones” :
[
{
”ZoneID” :3663,
```

```json
”groups” :
[
{
”group” :1,
”name” : ”yellow” ,
”scenes” :
[
{
”scene” :5,
”name” : ”demo￿scene”
}
]
},
{
”group” :2,
”name” : ”gray” ,
”scenes” :[]
}
]
}
]
}
}
```

10.2.2
------

query2

Differs from query(1) only in the format of the the returned json struct.

**Synopsis**

```json
HTTP GET /json/Property/query2?query=/Folder1(Property1,Property2)/Folder2(Property1) HTTP GET
```

/json/Property/query2?query=/Folder1/Folder2/Folder3(Property1) HTTP GET /json/Property/query2?query=/Fo

Folder  selects the nodes to descend,  Property  declares which attributes we are extracting from the

current node. If no properties are declared for a folder, nothing is extracted, and the node will not show

up in the resulting json structure.

| Parameter | Description | Remarks |
|---|---|---|
| query | querystring | Mandatory |

query string Mandatory

**Response**

```json
HTTP Status 200
```

result.value string value of the property

**Sample**

```json
GET  ’/json/property/query2?query=/apartment/zones/*(scenes)/groups/*(name)/scenes/*(name)’
{
”ok” : true ,
”result” :{
”zone0” :{
...
},
”zone6268” :{
”group0” :{
”name” : ”broadcast”
```

```json
},
”group1” :{
”name” : ”yellow” ,
”scene5” :{
”name” : ”dining”
},
”scene6” :{
”name” : ”TV”
},
},
”group2” :{
”name” : ”gray” ,
”scene6” :{
”name” : ”TV”
},
”scene17” :{
”name” : ”blinds￿15%”
},
},
}
...
}
}
```

The difference to query1 format is, that zones/groups/scenes are not returned as arrays of elements,

but each element individually as a named property. This more closely matches the query format and

facilitates accessing a specific element, e.g. zone6268.group1.scene6

Mind that the zones/groups folders are not part of the resulting json structure since no attributes is

extracted from them. We could re-add them to the output using the wildcard (*) property match

Different from query1, we are not extracting the zoneid and scene name attribute, since that information

is already contained in the element name. Neither are there any empty scene arrays. This makes the

resulting json structure quite a bit smaller and easier read by a human. Potentially the json structure

uses less memory is faster to generate, transfer, parse and render by a web application
