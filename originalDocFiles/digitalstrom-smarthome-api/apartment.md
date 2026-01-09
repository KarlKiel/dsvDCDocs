# Digitalstrom REST-based Smarthome API

## GET APARTMENT

#### Syntax:
```
GET {{host}}/api/v1/apartment
Authorization: Bearer {{api-key}}
```
#### Paramter:
```
include
```
optional

#### Options:
- installation
- dsDevices
- submodules
- functionBlocks
- zones
- clusters
- applications
- dsServer
- controllers
- apiRevision
- meterings

#### Example:
```
GET {{host}}/api/v1/apartment?include=installation,dsDevices,submodules,functionBlocks,zones,clusters,applications,dsServer,controllers,apiRevision,meterings
Authorization: Bearer {{api-key}}
```

## 
## GET APARTMENT STATES

#### Syntax:
```
GET {{host}}/api/v1/apartment/status
Authorization: Bearer {{api-key}}
```
optional

#### Options:
- installation
- dsDevices
- submodules
- functionBlocks
- zones
- clusters
- applications
- dsServer
- controllers
- apiRevision
- meterings

#### Example:
```
GET {{host}}/api/v1/apartment/status?include=installation,dsDevices,submodules,functionBlocks,zones,clusters,applications,dsServer,controllers,apiRevision,meterings
Authorization: Bearer {{api-key}}
