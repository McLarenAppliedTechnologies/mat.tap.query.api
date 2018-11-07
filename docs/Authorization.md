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

# Authorization

This API uses a Token authentication. You use your user/password to ask the server for an access token, then you can include this access token in the header in all requests to the server.

## Getting a token

To ask for a token you have to use the following endpoint filling up the parameters ```username```, ```password``` and ```grant_type``` in the body of the request:

```
GET /token
```

Note that default ```username``` and ```password``` in a new installation of the API are **"admin"**, **"admin"** respectively. Field ```grant_type``` is always the same value **"password"**.

Following image shows an example of this type of request:

<img src="Authorization1.png" alt="drawing" width="80%"/>

Field ```access_token``` of the result gives you the bearer token that you want to use. Token expires in the number of seconds returned in JSON (1 day here).

## Authorization of request

After getting a token from the server you have to use it in each request of the API putting it as a parameter in the Header of each call. The parameter name to use is ```Authorization``` and the value is keyword **"bearer"** followed by a space and the **Token** string.

<img src="Authorization2.png" alt="drawing" width="80%"/>

## Authorization using Swagger UI

If you are using Swagger UI for test the API you can ask for a token accessing to the  **Authentication** controller in the own interface of the Swagger UI main page. You only need to fill up the fields ```username``` and ```password``` and press the button **"Try it out"**:

![](/docs/SwaggerAuthentication.png)

The resulting field ```access_token``` gives you the bearer token that you must use in the rest of endpoints of the Swagger UI. All the endpoints that need authorization ask you for filling up an additional field named ```Authorization```. You must to put in this field the keyword **"bearer"** followed by a space and the **Token** string provided for the previous call to the **Authentication** controller.

Following images shows an example of how to use it:

![](/docs/SwaggerAuthorization.png)






