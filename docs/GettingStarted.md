# ![logo](/docs/branding.bmp) Telemetry Analytics API

### Table of Contents
- [**Introduction**](/README.md)<br>
- [**Getting started**](/docs/GettingStarted.md)<br>
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

	- An alternative way to test this API is using Swagger UI that it's embedded in the own implementation of this service. You can check more information about in the following [Swagger](#swagger) section of this documentation.
	
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

The SQLRace connections that you can get from the endpoint ```api/connections``` are shared with server local SQLRace configuration. If you are working locally with this API service you can configure these connections accessing to ATLAS 10 or SQLRace client. You can access connections configuration from Atlas 10 via Options -> Database Connection Manager:

<img src="/docs/AtlasDbManager.png" alt="Atlas Database Manager" width="90%"/>


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

Telemetry Analytics API natively support the **Open API** (formerly Swagger) API documentation format. It also integrates a version of **Swagger UI**, a nice tool to display the API documentation in a user friendly way. You can access to Swagger UI implementation using the following url from a web browser:

```
http://hostname:port/swagger
```

![](/docs/SwaggerUI.png)

Swagger is a specification for documenting REST API. It specifies the format (URL, method, and representation) to describe REST web services. 

Swagger is a language-agnostic specification, with its declarative resource specification, clients can easily understand and consume services without any prior knowledge of server implementation or access to the server code.


