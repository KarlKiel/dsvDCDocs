1 Introduction
==============

Introduction

All requests are sent using HTTP GET and parameters added to the query string url like:

```json
/json/apartment/setName?name= ”My￿digitalStrom￿Server” &username=dssadmin&password=secret
```

If not properly authenticated the HTTP Status 403 is returned and the error response contains:

```json
{
”ok” : false ,
”message” : ”not￿logged￿in”
}
```

If an unknown method is requested the error message ”Unhandled Function” is returned:

```json
{
”ok” :  false ,
”message” :  ”Unhandled￿function”
}
```

If a request has been successfully processed the JSON answer contains an ”ok” and an optional ”result”

field. The result array is explained in the particular sections.

result array of result values

Where Group Names are allowed the following table lists the possible names.

Name Group Id Description

yellow 1
