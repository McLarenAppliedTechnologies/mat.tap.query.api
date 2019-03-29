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

# Authorization

This API uses token authentication. You use your username/password to ask the server for an access token, then you include this access token in the header in all requests to the server. Please note that for SqlRace, OAuth server is embedded. Hence, you can use the same API to get the access token as TAPI. However, for InfluxDb API, OAuth server is called Identity Server and is deployed as a separate API. Hence, the address of the token endpoint and Swagger UI should use the hostname and port number of that server. Please refer to the Swagger UI for [Identity Server](/docs/IdentityServer.md) to test the token endpoint. 

## Getting Access Token

To ask for a token you have to use the following endpoint with parameters `username`, `password` and `grant_type` (and `client_id` for Identity Server) in the body of the request as url encoded form data:

```
POST /token
```

Note that default values for ```username```, ```password``` and ```client_id``` in a new installation of the API are **"admin"**, **"admin"** and **"default.tapi.client"** respectively. Field ```grant_type``` is always the same value as **"password"**.

Following image shows an example of this type of request for the embedded OAuth Server:

<img src="Authorization1.png" alt="drawing" width="80%"/>

Following image shows an example of this type of request for Identity Server using InfluxDb as storage:

<img src="AuthorizationInflux.png" alt="drawing" width="80%"/>

Field ```access_token``` of the result gives you the bearer token that you want to use. Token expires in the number of seconds returned in JSON. If you have configured the client to allow using a refresh token, you may also see the field ```refresh_token``` in the response. More information on how to use the refresh token can be found in [Using Refresh Token](#using-refresh-token) section.

## Validate Access Token

In order to test the token you just obtained with Postman, try getting the sessions in the database using `/api/v1/connections/{connection name}/sessions`. Before making the GET request, navigate to **Authorization** tab in Postman, under **TYPE** dropdown, select **Bearer Token** and paste the access token you obtained from the token endpoint in the textbox next to **Token** as shown below. If the token is valid, you will get `200 OK` response and will be able to see sessions in the database if there is any. If the token is invalid, you will receive `401 Unauthorized` response:

<img src="validate_token_postman_new_api.png" alt="drawing" width="80%"/>

Equivalently, you can navigate to **Headers** tab in Postman and enter **Authorization** as **Key** and keyword "bearer" followed by a space and the Token string as **Value**:

<img src="Authorization2.png" alt="drawing" width="80%"/>

In order to test authentication using Swagger UI, please see [Testing Authentication with Swagger UI](/docs/SwaggerUIAuth.md).

## Using Refresh Tokens

Refresh token allows the client to continue using the Api in the absence of resource owner. By hitting the token endpoint with the refresh token periodically, access token continues to remain valid. To refresh the token, use the parameters ```client_id``` (your client id), ```grant_type``` (set to ```refresh_token```) and ```refresh_token``` (set the value of your refresh token) in the body of the request as url encoded form data.

```
POST /token
```

Note that unless otherwise configured, your refresh token will change each time you refresh the token. More information about configuring refresh token, refer to [Identity Server](/docs/IdentityServer.md/#create-new-client) section.







