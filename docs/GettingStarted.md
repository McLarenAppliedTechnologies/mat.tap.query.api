# ![logo](/docs/branding.bmp) Telemetry Analytics API

![Build Status](https://mat-ocs.visualstudio.com/Telemetry%20Analytics%20Platform/_apis/build/status/MAT.TAP.TelemetryAnalytics.API/MAT.TAP.TelemetryAnalytics.API%20-%20Pull%20Request%20Gateway?branchName=develop)

### Table of Contents
- [**Introduction**](/README.md)<br>
- [**Installation**](/docs/Installation.md)<br>
- [**Getting started**](/docs/GettingStarted.md)<br>
- [**Identity Server**](/docs/IdentityServer.md)<br>
- [**Authorization**](/docs/Authorization.md)<br>
- [**Querying Metadata**](/docs/Metadata.md)<br>
- [**Consuming Data**](/docs/ConsumingData.md)<br>
- [**Views**](/docs/Views.md)<br>

# Getting started

1. For querying the server it is recommended (but not essential) to use [Postman](https://getpostman.com).

	 - Download and install it if you don't have it.
	 - You don't need an account

2. Get a password.

	- For an example of this see the [Authorization](Authorization.md) page (but change the username and password)
	- For those who are not familiar with this, you use your password to ask the server for an access token. You then include this access token in the header in all requests to the server.

3. Use Swagger UI.

	- An alternative way to test this API is using Swagger UI that it's embedded in the own implementation of the server. You can check more information about it in the following [Swagger](#swagger) section of this documentation.
	
## Connections

Telemetry Query API could be connected to multiple sources of data. You can query these sources with the following endpoint:

```
GET api/connections
```

Result:

```json
[
    {
        "FriendlyName": "Season2017",
        "ServerName": "SqlRace-RESTAPI\\SQLEXPRESS",
        "IsSqlite": false,
        "DatabaseName": "SQLRACE01",
        "Identifier": "Season2017",
        "RootPath": "C:\\SQLRace_Filestream"
    },
    {
        "FriendlyName": "Live Session Cache",
        "ServerName": "DbEngine=SQLite;Data Source=C:\\Users\\AdminRestApi\\AppData\\Local\\McLaren Applied Technologies\\ATLAS 10\\SQL Race\\LiveSessionCache.ssn2;PRAGMA journal_mode=WAL;",
        "IsSqlite": true,
        "DatabaseName": "C:\\Users\\AdminRestApi\\AppData\\Local\\McLaren Applied Technologies\\ATLAS 10\\SQL Race\\LiveSessionCache.ssn2 ",
        "Identifier": "Live Session Cache",
        "RootPath": null
    }
]
```


## Where can I setup my connections?

### SqlRace Connections

The SQLRace connections that you can get from the endpoint ```api/connections``` are shared with server local SQLRace configuration. If you are working locally with this API service you can configure these connections accessing to ATLAS 10 or SQLRace client. You can access connections configuration from Atlas 10 via Options -> Database Connection Manager:

<img src="/docs/AtlasDbManager.png" alt="Atlas Database Manager" width="90%"/>

### InfluxDb Connections

The InfluxDb connections that you can get from the endpoint ```api/connections``` depend on the [InfluxDb Writer](https://github.com/McLarenAppliedTechnologies/mat.tap.aas.influxdb) configuration settings used while recording the session. These settings needs to be created in the `api/connections` before querying.

You can create a new connection using `POST` request:

```
POST api/connections
```

Request:

```
{
  "influxDbUrl": "http://10.228.4.17:8086",
  "measurementName": "Marple",
  "identifier": "Season2017",
  "sqlServerConnectionString": "server=10.228.5.4\\SQLEXPRESS;Initial Catalog=TelemetryAnalytics.Api.Sessions;User Id=test;Password=test;"
}
```

Result:

```
{
  "influxDbUrl": "http://10.228.4.17:8086",
  "measurementName": "Marple",
  "identifier": "Season2017",
  "sqlServerConnectionString": "server=10.228.5.4\\SQLEXPRESS;Initial Catalog=TelemetryAnalytics.Api.Sessions;User Id=test;Password=test;"
}
```

- `influxDbUrl` is the address of the InfluxDb instance used to store data.
- `measurementName` is the name of the InfluxDb database (timeseries). Usually, this is the same as the topic name of the stream.
- `identifier` is string that uniquely identifies the connection.
- `sqlServerConnectionString` is the connection details of the database that stores the session metadata.

You can also update and delete existing connections using `PUT` and `DELETE` requests based on the connection identifier. Please refer to the [Swagger UI](#swagger) for more information.

## Query connection

Every connection provides a list of [Sessions](/docs/Sessions.md). You can query these [Sessions](/docs/Sessions.md) using the "FriendlyName" of the connection:

```
GET api/connections/{connection friendly name}/sessions
```
Example:
```
GET api/connections/Simulator/sessions
```
Paging is supported by this query. Page and page size could be specified. Default page size is 50 sessions in one page.

```
GET api/connections/Simulator/sessions?page=1&pageSize=
```
Result:
```json
    {
        "Key": {
            "value": "67ad8a93-6ba7-4c5d-9947-9c0e963ccf9e"
        },
        "Identifier": "171023220758",
        "TimeOfRecording": "2017-10-23T22:07:58.3646176",
        "TimeZone": "UTC",
        "SessionType": "SessionType",
        "StartTime": "00:00:00",
        "EndTime": "00:00:00",
        "LapsCount": 0,
        "Id": 1301,
        "State": 2
    }, {
        "Key": {
            "value": "0661d026-271d-4f2a-a255-b977438a10fe"
        },
        "Identifier": "171023220700",
        "TimeOfRecording": "2017-10-23T22:07:00.0236738",
        "TimeZone": "UTC",
        "SessionType": "SessionType",
        "StartTime": "00:00:00",
        "EndTime": "00:00:00",
        "LapsCount": 0,
        "Id": 1296,
        "State": 2
    }
```

## Swagger

Telemetry Analytics API natively support the **Open API** (formerly Swagger) API documentation format. It also integrates a version of **Swagger UI**, a nice tool to display and test all the API endpoints in a user friendly way. You can access to Swagger UI implementation using the following url from a web browser:

```
http://{hostname}:{port}/swagger
```

![](/docs/SwaggerUI.png)

Swagger is a specification for documenting REST API. It specifies the format (URL, method, and representation) to describe REST web services. 

Swagger is a language-agnostic specification, with its declarative resource specification, clients can easily understand and consume services without any prior knowledge of server implementation or access to the server code.

You can query or consume an updated Open API / Swagger specification of the API accessing to the following url of the server:

```
http://{hostname}:{port}/swagger/docs/v1
```

Example

```json
{
  "swagger": "2.0",
  "info": {
    "version": "v1",
    "title": "Telemetry Analytics API"
  },
  "host": "127.0.0.1:8011",
  "schemes": [
    "http"
  ],
  "paths": {
    "/api/connections": {
      "get": {
        "tags": [
          "Connections"
        ],
        "summary": "Gets page of configured storage connections.",
        "operationId": "Connections_GetAllConnections",
        "consumes": [
          
        ],
        "produces": [
          "application/json",
          "text/json",
          "application/xml",
          "text/xml"
        ],
        "responses": {
          "200": {
            "description": "OK",
            "schema": {
              "$ref": "#/definitions/Object"
            }
          }
        },
        "deprecated": false
      }
    },
    
    
```

