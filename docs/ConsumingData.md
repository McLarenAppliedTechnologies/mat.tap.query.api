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


Consuming Data
=====================

There are multiple ways to consume data and multiple ways to filter them, all of them grouped under the common endpoint ```/data```. Parameters of the session to query could be specified in URL or by a view. (See [Views](/docs/Views.md)).

### Base url masks

For InfluxDb data for querying with frequency specified:

```
GET api/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter}/{frequency}/data
```

And querying without specifying frequency:

```
GET api/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter}/{frequency}/data
```

Please note that you can only query one parameter at a time from InfluxDb at the moment. Support for multiple parameters is in the roadmap. 

All the resources under this base url mask are also available for SqlRace data. In addition, for SqlRace, you have the option to query data using the below base url mask (`/table/data`) which has support for querying data from multiple sessions and parameters. Readon for concrete examples.

For SqlRace data: 

```
GET api/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter1,parameter2,...,parameter_n}/table/{frequency}/data
```

### Url parameters

| Parameter name | Description                                                                                         | Default value | Example     |
|----------------|-----------------------------------------------------------------------------------------------------|---------------|-------------|
| connection     | Connection friendly name.   |               | SQLRACE01                                  |
| sessionKey     | Session Key.               |               | 016fa61e-33e2-7e85-1bc9-4ab56c668136       |
| parameters     | List of fields of the session to query data.  |               | vCar,NGear    |
| frequency      | Frequency of the results in Hz.  |               | 10            |

### Optional parameters

| Parameter name | Description                                                                                         | Default value | Example     |
|----------------|-----------------------------------------------------------------------------------------------------|---------------|-------------|
| from           | Filters data in session by time. All data returned would have time stamp after specified time.      |    `null`     | 10:34       |
| to             | Filters data in session by time. All data returned would have time stamp before specified time.     |    `null`     | 10:50       |
| lap            | Filters data for specified lap in session.                                                          |    `null`     | 3           |
| filter         | Generic filter expression describing filters.                                                       |    `null`     | vCar;ge;300 |
| page           | Index of page returned in result (0 is first page)                                                  |      0        | 3           |
| pageSize       | Size of one page.                                                                                   |     200       | 50          |

### Filter

Filter is an optional parameter that we can use to filter when consuming data. There are multiple types of filters could be specified.

#### Filter types

| Shortcut | Full name             |
|----------|-----------------------|
| eq       | Equal                 |
| gt       | Greater than          |
| ge       | Greater than or equal |
| lt       | Less than             |
| le       | Less than or equal    |
| ne       | Not equal             |

#### Parameter filter

Structure of one parameter filter instance is:

Parameter filter url mask
```
{parameterName};{filterOperationShortcut};{value}
```

Parameter filter example
```
vCar;gt;300
```

Example of filtering values that have vCar between 100 and 150 kph. 

Filter setting example
```
vCar:Chassis;gt;100,vCar:Chassis;le;150
```

Note that some paremeters have a colon ( : ) in, so we use semicolons ( ; ) for the filtering.

Flat data
---------

The endpoint ```/data``` is the base endpoint for consuming data in the API. It will return all samples flat across all time ranges. This view of data is paged. Default page size is 200 samples.

Mask
```
GET api/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter1,parameter2,...,parameter_n}/{frequency}/data
```

Example
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/parameters/vCar/10/data?from=11:15&to=11:20&filter=vCar;gt;100,vCar;le;150
```

Result
```json
    {
        "Time": "11:19:45.0450000",
        "Values": {
            "vCar": 149.88
        }
    },
    {
        "Time": "11:19:45.0350000",
        "Values": {
            "vCar": 149.28
        }
    },
    {
        "Time": "11:19:45.0250000",
        "Values": {
            "vCar": 149.45
        }
    },
    {
        "Time": "11:19:45.0150000",
        "Values": {
            "vCar": 149.3
        }
    },
    {
        "Time": "11:19:45.0050000",
        "Values": {
            "vCar": 148.35
        }
    }
```

Number of samples
-----------------

Easiest and fastest way to start working with a specific query of data is using the ```/data/count``` endpoint that returns
the number of samples that fit all filters. Note: instead of parameters you can use [Views](/docs/Views.md).

Mask
```
GET api/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter1,parameter2,...,parameter_n}/{frequency}/data/count
```

Example
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/parameters/vCar/10/data/count?from=11:15&to=11:20&filter=vCar;gt;100,vCar;le;150
```

Result
```
5646
```

Time ranges
-----------

The ```/data/timeRanges``` endpoint returns time ranges of result samples. Note that you can use all the parameters and filters used in the rest of endpoints.

Mask
```
GET api/connections/{connection name}/sessions/{sessionKey}
/parameters/{parameter1,parameter2,...,parameter_n}/{frequency}/data/timeRanges
```

Example
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c
/parameters/vCar/10/data/timeRanges?from=11:15&to=11:20&filter=vCar;gt;100,vCar;le;150
```

Result
```json
    {
        "StartTime": "11:19:41.1650000",
        "EndTime": "11:19:45.0550000",
        "Span": "00:00:03.8900000"
    },
    {
        "StartTime": "11:19:33.8350000",
        "EndTime": "11:19:35.1250000",
        "Span": "00:00:01.2900000"
    },
    {
        "StartTime": "11:19:28.6250000",
        "EndTime": "11:19:29.7050000",
        "Span": "00:00:01.0800000"
    },
    {
        "StartTime": "11:19:08.6650000",
        "EndTime": "11:19:09.8150000",
        "Span": "00:00:01.1500000"
    },
    {
        "StartTime": "11:18:51.1550000",
        "EndTime": "11:18:54.6950000",
        "Span": "00:00:03.5400000"
    }
```    

Grouped data
------------

The ```/data/grouped``` endpoint will return data grouped to time ranges. Note that you can use all the parameters and filters used in the rest of endpoints.

Mask
```
GET api/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter1,parameter2,...,parameter_n}/{frequency}/data/grouped
```

Example
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/parameters/vCar/10/data/grouped?from=11:15&to=11:20&filter=vCar;gt;100,vCar;le;150
```

Result
```json
{
        "StartTime": "11:19:41.1650000",
        "EndTime": "11:19:45.0550000",
        "Span": "00:00:03.8900000",
        "Values": [
            {
                "Time": "11:15:17.1750000",
                "Values": {
                    "vCar": 149.97
                }
            },
            {
                "Time": "11:15:17.1650000",
                "Values": {
                    "vCar": 149.18
                }
            },
            {
                "Time": "11:15:17.1550000",
                "Values": {
                    "vCar": 149.17
                }
            },
            {
                "Time": "11:15:17.1450000",
                "Values": {
                    "vCar": 149.67
                }
            },
            {
                "Time": "11:15:17.1350000",
                "Values": {
                    "vCar": 149.66
                }
            }
		]
}
```

Multiple sessions data
----------------------

All data queries could be done over multiple sessions for SqlRace data (InfluxDb data support for this feature is in the roadmap).

Mask
```
GET api/connections/{connection name}/sessions/{sessionKey1,sessionKey2,...,sessionKeyN}/parameters/{parameter1,parameter2,...,parameter_n}/table/{frequency}/data
```

Example
```
GET api/connections/M800960/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c,asdsad199-16155a-43ec-bb0a-12sadsa23a
/parameters/vCar/table/10/data?from=11:15&to=11:20&filter=vCar:Chassis;gt;100,vCar:Chassis;le;150
```

Result
```json
[
    {
        "Key": "e5d51619-13df-4cb8-b263-940be28aae8f",
        "Value": [
            {
                "StartTime": "12:18:54.6550000",
                "EndTime": "12:18:57.7550000",
                "Span": "00:00:03.1000000"
            },
            {
                "StartTime": "12:21:09.7550000",
                "EndTime": "12:21:15.2550000",
                "Span": "00:00:05.5000000"
            },
            {
                "StartTime": "12:23:20.4550000",
                "EndTime": "12:23:25.7550000",
                "Span": "00:00:05.3000000"
            },
            {
                "StartTime": "12:25:41.1550000",
                "EndTime": "12:25:47.4550000",
                "Span": "00:00:06.3000000"
            },
            {
                "StartTime": "12:27:55.4550000",
                "EndTime": "12:28:01.5550000",
                "Span": "00:00:06.1000000"
            },
            {
                "StartTime": "12:30:04.9550000",
                "EndTime": "12:30:08.5550000",
                "Span": "00:00:03.6000000"
            }
        ]
    },
    {
        "Key": "fd309247-66db-41cf-87b9-4760902f3981",
        "Value": [
            {
                "StartTime": "11:32:21.2970000",
                "EndTime": "11:32:25.0970000",
                "Span": "00:00:03.8000000"
            },
            {
                "StartTime": "11:33:43.6970000",
                "EndTime": "11:33:47.7970000",
                "Span": "00:00:04.1000000"
            },
            {
                "StartTime": "11:37:45.8970000",
                "EndTime": "11:37:49.2970000",
                "Span": "00:00:03.4000000"
            },
            {
                "StartTime": "11:39:42.8970000",
                "EndTime": "11:39:46.4970000",
                "Span": "00:00:03.6000000"
            },
            {
                "StartTime": "11:42:09.2970000",
                "EndTime": "11:42:13.4970000",
                "Span": "00:00:04.2000000"
            },
            {
                "StartTime": "11:43:27.3970000",
                "EndTime": "11:43:31.4970000",
                "Span": "00:00:04.1000000"
            },
            {
                "StartTime": "11:45:36.6970000",
                "EndTime": "11:45:41.0970000",
                "Span": "00:00:04.4000000"
            },
            {
                "StartTime": "11:48:21.7970000",
                "EndTime": "11:48:26.8970000",
                "Span": "00:00:05.1000000"
            },
            {
                "StartTime": "11:50:41.2970000",
                "EndTime": "11:50:44.3970000",
                "Span": "00:00:03.1000000"
            }
        ]
    }
]
```
