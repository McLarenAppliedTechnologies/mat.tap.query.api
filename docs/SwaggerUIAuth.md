# ![logo](/docs/branding.png) Telemetry Analytics API

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

## Testing Authentication with Swagger UI

### SqlRace API

If you are using Swagger UI to test the API you can ask for a token accessing to the  **Authentication** controller in the own interface of the Swagger UI main page. You only need to fill up the fields ```username``` and ```password``` and press the button **"Try it out"**:

![](/docs/SwaggerAuthentication.png)

The resulting field ```access_token``` gives you the bearer token that you must use in the rest of endpoints of the Swagger UI. All the endpoints that need authorization ask you for filling up an additional field named ```Authorization```. You must to put in this field the keyword **"bearer"** followed by a space and the **Token** string provided for the previous call to the **Authentication** controller.

Following images shows an example of how to use it:

![](/docs/SwaggerAuthorization.png)

### Identity Server

If you are using TAPI with InfluxDb, you need to navigate to `/swagger` in Identity Server. In the `Authentication` controller, click on **Try it out**, enter the user credentials and click on **Execute**:

![](/docs/Swagger_Influx.png)

You can verify the access token with any of the available controllers in Swagger UI. At the top of the page, click on **Authorize**, enter **"Bearer"**, followed by a space and paste the access token from the previous request and click on **Authorize** as shown in the image. You only need to set token once per session with Swagger UI. It will automatically be incldued in the headers for subsequent requests unless you logout using the **Logout** button in **Authorize** or the token expires.

![](/docs/bearer_token_influx.png)

Now, try using any of controllers. For example, try getting all the users using `/api/v1/users`. If the token authentication is successful, you will see the list of users allowed for TAPI. Otherwise, you will get `401 Unauthorized` as response.

![](/docs/swagger_token_validation.png)

You can use the same token and and steps above with TAPI to test controllers in TAPI's own Swagger UI.