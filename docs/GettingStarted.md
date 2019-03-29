# ![logo](/docs/branding.bmp) Telemetry Analytics API

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
  - You don't need to create a Postman account to use it.

2. Get a token.
	- For an example of this, see the [Authorization](Authorization.md) page (but change the username and password)
	- For those who are not familiar with token authentication, you use your username and password to ask the authentication server for an access token. You then include this access token in the header in all HTTP requests to the API.

3. Use Swagger UI.
	- An alternative way to test this API is using Swagger UI that it's embedded in the server. Refer to [Swagger](#swagger) section of this documentation for more information on how to use it.

4. Data Explorer UI.
	- A web based application that allows to visualize data ingested from the API, providing a set of features like aggregations and filtering.
	
## Connections

Under connections it is possible to get, create, update or delete connections.

## Where can I setup my connections?

### SqlRace Connections

SQLRace connections retrieved via api/v1/connections are from the shared SQLRace configuration on the server. If you are working locally with this API service you can configure these connections using ATLAS 10 or SQLRace client. You can access connections configuration from Atlas 10 via Options -> Database Connection Manager:

<img src="/docs/AtlasDbManager.png" alt="Atlas Database Manager" width="90%"/>


Telemetry Query API could be connected to multiple sources of data. You can query these sources with the following endpoint:

```
GET api/v1/connections
```

Result:
```json
[
    {
        "ServerName": "SqlRace-RESTAPI\\SQLEXPRESS",
        "IsSqlite": false,
        "DatabaseName": "SQLRACE01",
        "Identifier": "Season2017",
        "RootPath": "C:\\SQLRace_Filestream"
    },
    {
        "ServerName": "DbEngine=SQLite;Data Source=C:\\Users\\AdminRestApi\\AppData\\Local\\McLaren Applied Technologies\\ATLAS 10\\SQL Race\\LiveSessionCache.ssn2;PRAGMA journal_mode=WAL;",
        "IsSqlite": true,
        "DatabaseName": "C:\\Users\\AdminRestApi\\AppData\\Local\\McLaren Applied Technologies\\ATLAS 10\\SQL Race\\LiveSessionCache.ssn2 ",
        "Identifier": "Live Session Cache",
        "RootPath": null
    }
]
```

### InfluxDb Connections

InfluxDb connections that you can get from the endpoint ```api/v1/connections``` depend on the [InfluxDb Writer](https://github.com/McLarenAppliedTechnologies/mat.tap.aas.influxdb) configuration settings used while recording the session. These settings need to be created in the `api/v1/connections` before querying.

### Description:

It is possible to store data for a given topic into multiple influx databases based on topic name and the label used in the connection settings.
This schema allows to scale the data horizontaly and separate it according to different labels.
The *sqlServerConnectionString* and *identifier* are shared between the *influxDbDetails*.

You can create a new connection using `POST` request:

```
POST api/v1/connections
```

Request:
```Json
{
  "influxDbDetails": [
    {
      "topicName": "Topic",
      "label": "Driver1",
      "influxDbUrl": "http://localhost:8000",
      "influxDbDatabase": "Database1",
      "measurementName": "Marple",
      "identifier": "Season2017"
    },
    {
      "topicName": "Topic2",
      "label": "*",
      "influxDbUrl": "http://localhost:8000",
      "influxDbDatabase": "Database1",
      "measurementName": "Furnels",
      "identifier": "Season2017"
    }
  ],
  "identifier": "Season2017",
  "sqlServerConnectionString": "server=.\\SQLEXPRESS;Initial Catalog=Database;User Id=UserId;Password=Password;"
}
```

Result:
```Json
[
  {
    "influxDbDetails": [
      {
        "topicName": "Topic1",
        "label": "*",
        "influxDbUrl": "http://localhost:8000",
        "influxDbDatabase": "Database1",
        "measurementName": "Marple",
        "identifier": "Season2017"
      },
      {
        "topicName": "Topic2",
        "label": "*",
        "influxDbUrl": "http://localhost:8000",
        "influxDbDatabase": "Database1",
        "measurementName": "Furnels",
        "identifier": "Season2017"
      }
    ],
    "identifier": "Season2017",
    "sqlServerConnectionString": "server=.\\SQLEXPRESS;Initial Catalog=Database;User Id=UserId;Password=Password;"
  },
  *...*
```

- `topicName` is the name of the topic used to stream data.
- `label` label can be used to split sessions within a topic. *Note: If no label is specified than this will be used as the default one.*
- `influxDbDatabase` is the name of the influx db database.
- `influxDbUrl` is the address of the InfluxDb instance used to store data.
- `measurementName` is the name of the measurement used to store parameter samples within the influx database. Good practice to use topic name and label combination (without special characters), but it can be anything.
- `identifier` is a string that uniquely identifies the connection.
- `sqlServerConnectionString` is the connection string of the SQL database that stores session metadata.

You can also update and delete existing connections using `PUT` and `DELETE` requests based on the connection identifier. Please refer to the [Swagger UI](#swagger) for more information.

## Query sessions

Every connection provides a list of [Sessions](/docs/Sessions.md). You can query these [Sessions](/docs/Sessions.md) using the "FriendlyName" of the connection:

```
GET api/{apiVersion}/connections/{connection friendly name}/sessions
```
Example:
```
GET api/v1/connections/Simulator/sessions
```
Paging is supported by this query. Page and page size can be specified. Default page size is 50 sessions in one page.

```
GET api/v1/connections/Simulator/sessions
```

SqlRace response:

Result:
```json
[
  {
    "Key": "7a0f9cdf-8535-b0bd-5efc-83c5ea5e95a0",
    "Identifier": "Identfier1",
    "TimeOfRecording": "2017-05-12T10:42:15.2374652",
    "TimeZone": "UTC",
    "SessionType": "Session",
    "Start": "1970-01-01T13:12:54.425",
    "End": "1970-01-01T14:46:38.36",
    "LapsCount": 28,
    "State": "Historical",
    "SessionDetails": null
  },
  {
    "Key": "0bc99d7a-596f-c3e9-3454-fa4439891a8b",
    "Identifier": "Identfier2",
    "TimeOfRecording": "2017-05-11T14:04:29.8996347",
    "TimeZone": "UTC",
    "SessionType": "Session",
    "Start": "1970-01-01T12:40:59.949",
    "End": "1970-01-01T12:47:25.889",
    "LapsCount": 1,
    "State": "Historical",
    "SessionDetails": null
  }
]
```

InfluxDb response:

Result:
```json
[
  {
    "key": "0432c948-0ab0-4270-9fe9-d0b9b08af7a0",
    "identifier": "Identifier1",
    "timeOfRecording": "2016-03-28T14:23:09.3932003",
    "timeZone": null,
    "sessionType": "StreamSession",
    "start": "2019-03-28T18:22:14.771",
    "end": "2019-03-28T18:22:29.943",
    "lapsCount": 1,
    "state": "Closed",
    "topicName": "TopicName1",
    "sessionDetails": []
  },
  {
    "key": "0912bade-c473-4385-bb26-af0631148693",
    "identifier": "Identifier2",
    "timeOfRecording": "2016-03-28T15:12:39.4050654",
    "timeZone": null,
    "sessionType": "StreamSession",
    "start": "2019-03-28T19:12:37.756",
    "end": "2019-03-28T19:13:27.885",
    "lapsCount": 1,
    "state": "Closed",
    "topicName": "TopicName2",
    "sessionDetails": []
  }
]
```

## Swagger

Telemetry Analytics API natively supports the **Open API** (formerly Swagger) API documentation format. It also integrates a version of **Swagger UI**, a nice tool to display and test all the API endpoints in a user friendly way. You can access the Swagger UI using the following url from a web browser:

```
http://{hostname}:{port}/swagger
```

![](/docs/SwaggerUI.png)

Swagger is a specification for documenting RESTful APIs. It specifies the format (URL, method, and representation) to describe RESTful web services. 

Swagger is a language-agnostic specification, with its declarative resource specification, clients can easily understand and consume services without any prior knowledge of server implementation or access to the server code.

You can query or consume an updated Open API / Swagger specification of the API using the following url of the server:

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
