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


Querying Metadata
========  

There are multiple endpoints available to query sessions metadata information. In this section we are going to explain the different endpoints available and what information we can obtain from them.
  
Sessions  
========

The ```/sessions``` endpoint give you access to a list of **historical** sessions available for a given connection. You can filter this list of sessions by several parameters.

Optional parameters  
-------------------  
  
| Parameter name | Description | Default value | Example |  
|----------------|-------------------------------------------------------------|---------------|---------------------------------------------------------------------------|  
| details | This filter sessions by session details like driver, car etc. | | Driver:KHA,Car:P1GTR |  
| filter | This filter returned query with expression. | | LapsCount &gt; 5 %26%26 TimeOfRecording &gt; DateTime.Parse("2017-12-18") |  
| page | Index of page returned in result (0 is first page) | | 3 |  
| pageSize | Size of one page. | 200 | 50 |  
  
Filter  
------  
  
Please note that some characters are not allowed in URL and therefore must be specified with percentage and symbol number. For example: & must be replaced by %26.  
  
Mask
```
GET api/connections/{connection name}/sessions
```

Example  
```
GET api/connections/Simulator/sessions?items=Driver:KHA,Car:P1GTR&filter=LapsCount > 5 %26%26 TimeOfRecording >
DateTime.Parse("2017-12-18")
```

Result  
```json
[
    {
        "Key": "7b7b910b-0b3a-4aff-b64f-54204d8ea863",
        "Identifier": "171218111334",
        "TimeOfRecording": "2017-12-18T11:13:34.3866611",
        "SessionType": "Session",
        "StartTime": "12:46:02.0290000",
        "EndTime": "13:08:40.1190000",
        "LapsCount": 9,
        "Id": 1495,
        "State": "Historical"
    },
    {
        "Key": "652618d1-995e-48dd-91c3-600447f59d9e",
        "Identifier": "171218115852",
        "TimeOfRecording": "2017-12-18T11:58:52.8114764",
        "SessionType": "Session",
        "StartTime": "13:31:06.9320000",
        "EndTime": "13:42:57.5890000",
        "LapsCount": 6,
        "Id": 1497,
        "State": "Historical"
    }
]
```
  
Live sessions  
===================  

The ```/sessions/live``` endpoint give you access to a list of **live** sessions available for a given connection. You can filter this list of sessions by several parameters.

Optional parameters  
-------------------  
  
| Parameter name | Description | Default value | Example |  
|----------------|-------------------------------------------------------------|---------------|---------------------------------------------------------------------------|  
| filter | This filter returned query with expression. | | LapsCount &gt; 5 %26%26 TimeOfRecording &gt; DateTime.Parse("2017-12-18") |  
| page | Index of page returned in result (0 is first page) | | 3 |  
| pageSize | Size of one page. | 200 | 50 |  
  
Filter  
------  
  
Please note that some characters are not allowed in URL and therefore must be specified with percentage and symbol number. For example: & must be replaced by %26.  

Mask
```
GET api/connections/{connection name}/sessions/live
```
  
Example  
```
GET api/connections/Simulator/sessions/live
```


Parameters  
==========  

The ```/sessions/{sessionKey}/parameters``` endpoint give you access to a list **parameters** available for a specific session. The list of **parameters** of a session are the fields related to the data that we can finally consume in the section [Consuming Data](/docs/ConsumingData.md).

Mask
```
GET api/connections/{connection name}/sessions/{sessionKey}/parameters
```

Example  
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/parameters
```
  
Optional parameters  
-------------------  
  
| Parameter name | Description | Example |  
|----------------|----------------------------------------------------|---------|  
| page | Index of page returned in result (0 is first page) | 3 |  
| pageSize | Size of one page. | 50 |  
| contains | Text filter | vCar |  

### Paging  

Example  
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/parameters?page=2&pageSize=50
```
  
### Filtering  
  
Example 
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/parameters?contains=vCar
```
  
Result  
```json
{
        "Name": "vCar",
        "MaximumValue": 400,
        "MinimumValue": 0,
        "Description": "Car Speed"
    },
    {
        "Name": "vCarHigh",
        "MaximumValue": 400,
        "MinimumValue": 0,
        "Description": "Car speed from vHighestF adjusted to centreline"
    },
    {
        "Name": "vCarMG",
        "MaximumValue": 1000,
        "MinimumValue": 0,
        "Description": "vCar based on nEngine"
    },
    {
        "Name": "vCarEOS",
        "MaximumValue": 350,
        "MinimumValue": 0,
        "Description": "Car speed (end of straight)"
    },
    {
        "Name": "vCarLimited",
        "MaximumValue": 400,
        "MinimumValue": 0,
        "Description": "Power Limited Car Speed"
    },
    {
        "Name": "vCarMaxOilSample",
        "MaximumValue": 500,
        "MinimumValue": 0,
        "Description": "Max car speed in oil sample window"
    },
    {
        "Name": "vCarHighRaw",
        "MaximumValue": 400,
        "MinimumValue": 0,
        "Description": "Unlimited car speed for slip calcs from vCarHigh"
    }
```


Details  
=======

The ```/sessions/{sessionKey}/details``` endpoint give you access to a list **details** available for a specific session.

Mask
```
GET api/connections/{connection name}/sessions/{sessionKey}/details
```
  
Example  
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/details
```
  
Result  
```json
[
    {
        "Name": "Session Description",
        "Value": "P1GTR"
    },
    {
        "Name": "Session Name",
        "Value": "GAS R1"
    },
    {
        "Name": "Session Number",
        "Value": "R1"
    },
    {
        "Name": "Driver",
        "Value": "GAS"
    },
    {
        "Name": "Car",
        "Value": "P1GTR"
    },
    {
        "Name": "Circuit",
        "Value": "Bar"
    },
    {
        "Name": "Race/Test",
        "Value": "Simulator"
    },
    {
        "Name": "Pit Lane Trigger",
        "Value": "None"
    },
    {
        "Name": "Unit Data Source",
        "Value": "vTAG RF,Ethernet Telemetry/Wirelink "
    },
    {
        "Name": "Date of recording",
        "Value": "07/08/2017"
    }
]
```

Laps  
====  

The ```/sessions/{sessionKey}/laps``` endpoint give you access to the information with the **laps** of specific session. 
  
Mask
```
GET api/connections/{connection name}/sessions/{sessionKey}/laps
```
  
Example 
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/laps
```
  
Result
```json
[
    {
        "StartTime": "12:08:15.5120000",
        "EndTime": "12:09:05.7300000",
        "LapTime": "00:00:50.2180000",
        "CountForFastestLap": false,
        "Name": "Out Lap",
        "Number": 0
    },
    {
        "StartTime": "12:09:05.7300000",
        "EndTime": "12:13:37.1600000",
        "LapTime": "00:04:31.4300000",
        "CountForFastestLap": true,
        "Name": "Lap 16",
        "Number": 16
    },
    {
        "StartTime": "12:13:37.1600000",
        "EndTime": "12:16:15.0300000",
        "LapTime": "00:02:37.8700000",
        "CountForFastestLap": true,
        "Name": "Lap 17",
        "Number": 17
    },
    {
        "StartTime": "12:16:15.0300000",
        "EndTime": "12:18:48.8300000",
        "LapTime": "00:02:33.8000000",
        "CountForFastestLap": true,
        "Name": "Lap 18",
        "Number": 18
    }
]
```

    
