11 Database
===========

11
Database
11.1
Database Query
11.1.1
query
Returns data as a result of an SQL query.
Synopsis
HTTP GET /json/database/query
Parameter
Parameter Description
Remarks
database
name of the database Mandatory
sql
SQL query
Mandatory
Response
HTTP Status 200
result.data objects representing the data in the query response
Sample
GET /json/database/query?database=dsa&sql=select * from devices limit 1;
{
”result”:
{
”data”:
[
{
”key”: ”3504175fe00000000000000000017bf6003504175fe00000000000001000000e4f00”,
”dsuid”: ”3504175fe00000000000000000017bf600”,
”dsmdsuid”: ”3504175fe00000000000001000000e4f00”,
”deviceid”: ”219”,
”productid”: ”1224”,
”functionid”: ”33027”,
”version”: ”833”,
”zoneid”: ”3663”,
”configurationid”: ”255”
}
]
},
”ok”: true
}
167

