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


Consuming Data
=====================

This section describes what endpoints are available and how to consume parameter data.<br />
URIs for consuming data are grouped under the namespace `/data`.<br />
It is possible to consume data using the api from either InfluxDB or SqlRace, being possible to retrieve the data at raw rate or at a given frequency in either case.

- When using <ins>InfluxDb</ins> as the backing data storage, a wide variety of aggregations are supported, for example: Count, Mean, Median, Sum, First, Last, Max, Min, Stddev. *NOTE: Distinct is currently not supported for multiple parameters*
- When using <ins>SqlRace</ins> as the backing data storage, First, Mean, Min, Max aggregations are supported.
- The resource `/data/aggregate` is only supported by InfluxDB backing data storage.
- All resources under `/data` for both data storages provide different response formats. All endpoints support both json and csv, by specifying header Accept-Type as "application/json" or "text/csv", the later one being more optimized for downloading large amounts of parameter data. This flexible approach allows for a data format to be chosen given the technical requirements of the client application (e.g. web applications). Both data storages allow you to query one or multiple parameters for a given session.

## Consuming Parameter Data

It is possible to consume parameter data using `/data` endpoint. For resources under `/data` path, two url masks are available for querying data.


### Base url masks

Querying at raw rate:

```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter_1,parameter_2,...,parameter_n}/data
```

### Url parameters

| Parameter name | Description                                                                                         |    Example                                |
|----------------|-----------------------------------------------------------------------------------------------------|-------------------------------------------|
| connection     | Connection name.                                                                                    |    `SQLRACE01`                            |
| sessionKey     | Session Key.                                                                                        |    `016fa61e-33e2-7e85-1bc9-4ab56c668136` |
| parameter      | Name(s) of the parameter(s). *Multiple parameters can be requested by separating them with a comma* |    `vCar:Chassis, gLat:Chassis`           |

<br />

Query with frequency:

```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter_1;aggregation_1,...,parameter_n;aggregation_n};/{frequency}/data 
```

### Url parameters

| Parameter name | Description                                                                                                                  |    Default value    |    Example                                |
|----------------|------------------------------------------------------------------------------------------------------------------------------|---------------------|-------------------------------------------|
| connection     | Connection name.                                                                                                             |                     |    `SQLRACE01`                            |
| sessionKey     | Session Key.                                                                                                                 |                     |    `016fa61e-33e2-7e85-1bc9-4ab56c668136` |
| parameter      | Name(s) of the parameter(s). *Multiple parameters can be requested by separating them with a comma*                          |                     |    `vCar:Chassis, gLat:Chassis`           |
| aggregation    | Optional aggregation function separated by a semicolon `;`. *Do not add semicolon if you are not specifying an aggregation.* |    `mean`           |    `;max`                                 |
| frequency      | Frequency of the results in Hz.                                                                                              |                     |    10                                     |

<br />

### Optional parameters for both base urls

Optionally it is possible to filter the data by the following parameters:

| Parameter name | Description                                                                                                                         |    Default value    |    Example                               |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------|---------------------|------------------------------------------|
| from           | Filters data in session by time in microseconds or nanoseconds. All data returned would have a time stamp after the specified time. |    `null`           |    1546347600/1546347600000              |
| to             | Filters data in session by time in microseconds or nanoseconds. All data returned would have a time stamp before the specified time.|    `null`           |    1546348200/1546348200000              |
| lap            | Filters data for specified lap in session.                                                                                          |    `null`           |    3                                     |
| filter         | Generic filter expression describing filters.                                                                                       |    `null`           |    `vCar:Chassis;ge;300`                 |
| page           | Index of page returned in result (0 is first page)                                                                                  |      0              |    3                                     |
| pageSize       | Size of one page.                                                                                                                   |    `null`           |    50                                    |
| timeUnit       | Units of time. Use `Ns` for nanoseconds and `Ms` for microseconds.                                                                  |      `Ns`           |    `Ms`                                  |
| groupBy        | Groups queries by fields (Only supported for InfluxDb storage). Use `*` to group by all available fields.                           |    `null`           |    `groupBy=lapnumber&groupBy=sessionId` |

#### Optional parameters validation
As mentioned when requesting either raw data or frequency one, it is possible to provide a set of optional parameters. The API will validate the optional parameters provided and in case of failure in binding any of the parameters it will provide a response with a 422 status code (Unprocessable Entity) and a message or a set of messages with the failure errors. <br />
This validation layer is provided only for API using InfluxDb as data storage. The parameters filter and groupby are excluded from model validation.

##### Query with one wrong parameter

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis;mean,gLat:Chassis;mean/data?Lap=test
```

Json Result:
```Json
{
  "message": "Model state validation failed",
  "errors": [
    {
      "field": "Lap",
      "message": "The value 'test' is not valid for Lap."
    }
  ]
}
```

##### Query with multiple wrong parameter 

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis;mean,gLat:Chassis;mean/data?page=-1&pageSize=abc
```

Json Result:
```Json
{
  "message": "Model state validation failed",
  "errors": [
    {
      "field": "Page",
      "message": "Please enter a positive number"
    },
    {
      "field": "PageSize",
      "message": "The value 'abc' is not valid for PageSize."
    }
  ]
}
```

### More on filtering

Filter currently is done upon applying to a specific field the sample mode <ins>Mean</ins>. There are multiple types of filters for example:

#### Filter types

| Shortcut | Full name             |
|----------|-----------------------|
| eq       | Equal                 |
| gt       | Greater than          |
| ge       | Greater than or equal |
| lt       | Less than             |
| le       | Less than or equal    |
| ne       | Not equal             |

#### Parameter filter construction

Structure of one parameter filter is:

Parameter filter url mask
```
{parameterName};{filterOperationShortcut};{value}
```

Parameter filter example
```
vCar:Chassis;gt;300
```

Example of filtering values that have vCar between 100 and 150 kph. 

Filter setting example
```
vCar:Chassis;gt;100,vCar:Chassis;le;150
```

Note that some parameters have a colon ( : ) in, so we use semicolons ( ; ) for the filtering.

#### Query Parameter Data

Endpoint
```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter_1,parameter_2,...,parameter_n}/data
```

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis;mean,gLat:Chassis;mean/data?pageSize=10
```

Csv Result:
```
time,tagValues,vCar:Chassis,gLat:Chassis
1549534712083000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.2,NaN
1549534712093000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.2,NaN
1549534712103000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.15,NaN
1549534712113000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.11,NaN
1549534712123000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.11,NaN
1549534712133000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.15,NaN
1549534712143000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.21,NaN
1549534712153000000,sessionId=64192714-90fe-4865-a191-a6887e465184,1.26,NaN
1549534712163000000,sessionId=64192714-90fe-4865-a191-a6887e465184,0.46,NaN
1549534712167000000,sessionId=64192714-90fe-4865-a191-a6887e465184,NaN,0.025
```

Json Result:
```Json
{
  "parameters": {
    "vCar:Chassis": {
      "timestamps": [
        1549534712083000000,
        1549534712093000000,
        1549534712103000000,
        1549534712113000000,
        1549534712123000000,
        1549534712133000000,
        1549534712143000000,
        1549534712153000000,
        1549534712163000000
      ],
      "values": [
        1.2,
        1.2,
        1.15,
        1.11,
        1.11,
        1.15,
        1.21,
        1.26,
        0.46
      ]
    },
    "gLat:Chassis": {
      "timestamps": [
        1549534712167000000
      ],
      "values": [
        0.025
      ]
    }
  }
}
```

#### Query Parameter Data Frequency

## Query Parameter Data Frequency with a timestamp range

It is demonstrated in this section several requests and combinations of the filtering capabilities of the API. All combinations are possible and these are also available to be requested without frequency as well.

Endpoint
```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter_1;aggregation_1,parameter_2;aggregation_2,...,parameter_n;aggregation_n}/{frequency}/data
```

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis;mean,vCar:Chassis;max,gLat:Chassis;mean/10/data?pageSize=10&from=1549534714876000000&to=1549572500653000000
```

Csv Result:
```
time,tagValues,Mean(vCar:Chassis),Max(vCar:Chassis),Mean(gLat:Chassis)
1549534714800000000,,0,0,0.014
1549534714900000000,,0,0,-0.00525
1549534715000000000,,0,0,0.00475
1549534715100000000,,0,0,0.00625
1549534715200000000,,0,0,0.00475
1549534715300000000,,0,0,0.00075
1549534715400000000,,0,0,0.0045
1549534715500000000,,0,0,0.00575
1549534715600000000,,0,0,0.0085
1549534715700000000,,0,0,0.00575
```

Json Result:
```Json
{
  "timestamps": [
    1549534714800000000,
    1549534714900000000,
    1549534715000000000,
    1549534715100000000,
    1549534715200000000,
    1549534715300000000,
    1549534715400000000,
    1549534715500000000,
    1549534715600000000,
    1549534715700000000
  ],
  "values": {
    "Mean(vCar:Chassis)": [
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0
    ],
    "Max(vCar:Chassis)": [
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0,
      0.0
    ],
    "Mean(gLat:Chassis)": [
      0.013999999999999999,
      -0.0052499999999999995,
      0.004749999999999999,
      0.00625,
      0.004749999999999999,
      0.00075000000000000012,
      0.0045,
      0.0057500000000000008,
      0.0084999999999999989,
      0.005749999999999999
    ]
  }
}
```

## Query Parameter Data Frequency with a specific time unit (nanoseconds)

Endpoint
```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter_1;aggregation_1,parameter_2;aggregation_2,...,parameter_n;aggregation_n}/{frequency}/data
```

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis;mean,gLat:Chassis;mean/10/data?pageSize=10&timeUnit=ns
```

Csv Result:
```
time,tagValues,Mean(vCar:Chassis),Mean(gLat:Chassis)
1549534712000000000,,1.2,NaN
1549534712100000000,,0.762,0.00142857142857143
1549534712200000000,,1.214,0.002
1549534712300000000,,3.044,0.0055
1549534712400000000,,2.701,0.0065
1549534712500000000,,1.355,0.00375
1549534712600000000,,0.07,0.0005
1549534712700000000,,1.506,0.00075
1549534712800000000,,2.069,0.0015
1549534712900000000,,1.421,0.00175
```

Json Result
```Json
{
  "timestamps": [
    1549534712000000000,
    1549534712100000000,
    1549534712200000000,
    1549534712300000000,
    1549534712400000000,
    1549534712500000000,
    1549534712600000000,
    1549534712700000000,
    1549534712800000000,
    1549534712900000000
  ],
  "values": {
    "Mean(vCar:Chassis)": [
      1.2,
      0.7619999999999999,
      1.214,
      3.0440000000000005,
      2.7010000000000005,
      1.355,
      0.069999999999999993,
      1.5059999999999998,
      2.069,
      1.421
    ],
    "Mean(gLat:Chassis)": [
      "NaN",
      0.0014285714285714288,
      0.002,
      0.0055,
      0.0065000000000000006,
      0.00375,
      0.0004999999999999999,
      0.00075000000000000023,
      0.0014999999999999994,
      0.0017500000000000005
    ]
  }
}
```

## Query Parameter Data Frequency with a filter

Endpoint
```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter_1;aggregation_1,parameter_2;aggregation_2,...,parameter_n;aggregation_n}/{frequency}/data
```

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis;mean,gLat:Chassis;mean/10/data?pageSize=10&filter=vCar:Chassis;ge;300 
```

Csv Result:
```
time,tagValues,Mean(vCar:Chassis),Mean(gLat:Chassis)
1549534949900000000,,300.102,0.0414
1549534950000000000,,300.728,0.1048
1549534950100000000,,301.081,0.1378
1549534950200000000,,302.057,0.1194
1549534950300000000,,302.341,0.065
1549534950400000000,,302.818,0.0895
1549534950500000000,,303.211,-0.0263
1549534950600000000,,303.808,0.00690000000000001
1549534950700000000,,304.199,0.0133
1549534950800000000,,304.743,0.16
```

Json Result
```Json
{
  "timestamps": [
    1549534949900000000,
    1549534950000000000,
    1549534950100000000,
    1549534950200000000,
    1549534950300000000,
    1549534950400000000,
    1549534950500000000,
    1549534950600000000,
    1549534950700000000,
    1549534950800000000
  ],
  "values": {
    "Mean(vCar:Chassis)": [
      300.10200000000003,
      300.728,
      301.081,
      302.05700000000007,
      302.341,
      302.818,
      303.211,
      303.808,
      304.19900000000007,
      304.74300000000005
    ],
    "Mean(gLat:Chassis)": [
      0.041400000000000006,
      0.10480000000000002,
      0.13779999999999998,
      0.11940000000000003,
      0.065,
      0.0895,
      -0.026300000000000007,
      0.006900000000000006,
      0.013299999999999992,
      0.16000000000000003
    ]
  }
}
```

## Query Parameter Data Frequency with group by

Endpoint
```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter_1;aggregation_1,parameter_2;aggregation_2,...,parameter_n;aggregation_n}/{frequency}/data
```

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis/10/data?pageSize=1&groupby=lapnumber
```

Csv Result:
```
time,tagValues,Mean(vCar:Chassis),Mean(gLat:Chassis)
1549534712000000000,lapnumber=1l,1.2,NaN
1549535063400000000,lapnumber=2l,NaN,-0.6745
1549535063200000000,lapnumber=3l,222.256,NaN
1549535120200000000,lapnumber=4l,289.678571428571,0.764
1549535201500000000,lapnumber=5l,288.318888888889,1.18677777777778
1549535275900000000,lapnumber=6l,288.515,1.2145
1549535362100000000,lapnumber=7l,291.104,1.1925
```

Json Result
```Json
{
  "timestamps": [
    1549534712000000000,
    1549535063400000000,
    1549535063200000000,
    1549535120200000000,
    1549535201500000000,
    1549535275900000000,
    1549535362100000000
  ],
  "values": {
    "Mean(vCar:Chassis)": [
      1.2,
      "NaN",
      222.256,
      289.67857142857144,
      288.31888888888886,
      288.515,
      291.104
    ],
    "Mean(gLat:Chassis)": [
      "NaN",
      -0.6745,
      "NaN",
      0.764,
      1.1867777777777777,
      1.2144999999999997,
      1.1924999999999997
    ]
  },
  "tags": [
    {
      "lapnumber": "1l"
    },
    {
      "lapnumber": "2l"
    },
    {
      "lapnumber": "3l"
    },
    {
      "lapnumber": "4l"
    },
    {
      "lapnumber": "5l"
    },
    {
      "lapnumber": "6l"
    },
    {
      "lapnumber": "7l"
    }
  ]
}
```

#### Query Parameter Data Count

This functionality is available for `/data` with and without frequency.

Endpoint
```
GET api/v1/connections/{connection name}/sessions/{sessionKey}/parameters/{parameter};{aggregation}/{frequency}/data/count
```

Example
```
GET api/v1/connections/Connection/sessions/92ce7a51-83d1-43ec-bb0a-9cda685ca47c/parameters/vCar:Chassis;mean,gLat:Chassis;mean/10/data/count
```

Json Result
```Json
[
    {
        "name":"Mean(vCar:Chassis)",
        "count":6236.0
    },
    {
        "name":"Mean(gLat:Chassis)",
        "count":6291.0
    }
]
```

#### Aggregate Parameter Data over Tags.

If your backing storage is InfluxDb, you can aggregate parameter data over InfluxDb tags.

Endpoint
```
GET api/v1/connections/{connections}/sessions/{sessionKey}/parameters/{parameter};{aggregation}/data/aggregate?groupBy={tag1}&groupBy={tag2}
```

Example
```
GET api/v1/connections/Connection/sessions/64192714-90fe-4865-a191-a6887e465184/parameters/vCar:Chassis;mean,gLat:Chassis;mean/data/aggregate?groupBy=lapnumber
```

Csv Result
```
time,tagValues,vCar:Chassis,gLat:Chassis
1549534711876000000,lapnumber=1l,112.335688215992,0.18400058029484376
1549534711876000000,lapnumber=2l,199.965857988166,-0.82674907292954236
1549534711876000000,lapnumber=3l,225.134068751761,0.087585425383542262
1549534711876000000,lapnumber=4l,191.609365859654,0.2036408483625701
1549534711876000000,lapnumber=5l,209.018854306413,0.26592772152238897
1549534711876000000,lapnumber=6l,179.251327235393,0.25245055569364194
1549534711876000000,lapnumber=7l,186.329971860711,0.52127658846863922
```

Json Result
```Json
{
  "timestamps": [
    1549534711876000000,
    1549534711876000000,
    1549534711876000000,
    1549534711876000000,
    1549534711876000000,
    1549534711876000000,
    1549534711876000000
  ],
  "values": {
    "Mean(vCar:Chassis)": [
      112.335688215992,
      199.96585798816574,
      225.13406875176085,
      191.60936585965354,
      209.01885430641281,
      179.25132723539306,
      186.32997186071083
    ],
    "Mean(gLat:Chassis)": [
      0.18400058029484376,
      -0.82674907292954236,
      0.087585425383542262,
      0.2036408483625701,
      0.26592772152238897,
      0.25245055569364194,
      0.52127658846863922
    ]
  },
  "tags": [
    {
      "lapnumber": "1l"
    },
    {
      "lapnumber": "2l"
    },
    {
      "lapnumber": "3l"
    },
    {
      "lapnumber": "4l"
    },
    {
      "lapnumber": "5l"
    },
    {
      "lapnumber": "6l"
    },
    {
      "lapnumber": "7l"
    }
  ]
}
```
