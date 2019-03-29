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

# Identity Server

Identity Server implements OpenID Connect and provides authentication services for Telemetry Analytics API. Identity Server provides following services.

- Provides a RESTful API to manage TAPI users and clients;
- Facilitate OAuth2.0 flows to authorize access to TAPI resources.

### Deployment
#### .NET Core runtime
First you need to install .NET Core 2.1 runtime. You can download it [here](https://www.microsoft.com/net/download/dotnet-core/2.1). Example for Ubuntu 18.04 LTE: 

```
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt-get --yes install apt-transport-https
sudo apt-get update
sudo apt-get --yes install aspnetcore-runtime-2.1
```

#### Daemon installation
One way to run Identity Server is using systemd daemon service. In the release bundle, there is a shell script **daemon_deploy.sh** for daemon installation. 

Before you run it, execute following commands:
```
awk 'BEGIN{RS="^$";ORS="";getline;gsub("\r","");print>ARGV[1]}' daemon_deploy.sh
sudo chmod 777 daemon_deploy.sh
```

Then run:
```
./daemon_deploy.sh
```

You can verify it by:

```
journalctl --unit MAT.TAP.IdentityServer.service --follow -n 100
```

or start and stop by 

```
sudo systemctl stop MAT.TAP.IdentityServer.service
sudo systemctl start MAT.TAP.IdentityServer.service
```

or configure your config in **/opt/MAT.TAP.IdentityServer/appsettings.Production.json**

#### Running Identity Server from Terminal

Alternatively, you can use Identity Server by adding the relevant configuration in `appsettings.Production.json` file and running the following command from the terminal. Make sure you are in the same directory as the **MAT.TAP.IdentityServer.dll**.

    dotnet MAT.TAP.IdentityServer.dll --urls="http://*:5000"

**Please note that the daemon is using configuration file from /opt/MAT.TAP.IdentityServer/appsettings.Production.json and that service needs restart to reconfigure.**

A sample configuration and an explanation of settings is given below.

```
{
  "ConnectionStrings": {
    "IdentityDatabase": "server=(localdb)\\MSSQLLocalDB;Initial Catalog=MAT-TAP-IdentityServer"
  },
  "InitializeDatabase": true,
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  },
  "OAuthServer": "http://localhost:5000",
  "TokenCleanUp": {
    "EnableCleanUp": true,
    "CleanUpInterval": 10080 // In minutes.,
  },
  "AllowedOrigins": [
    "*"
  ] // Cors Policy. List of origins or wildcard character to allow all origins.
}
```

- `OAuthServer`: Address of the Identity Server to authorize access to user management Apis. **If you are accessing the API from outside using an external IP address you may need to use that external address here.**
- `InitializeDatabase`: Set `True` to initialize database configured in ConnectionStrings section. This creates the database if it doesn't exist, applies any pending database migrations and creates the admin user and a default client.
- `ConnectionStrings:IdentityDatabase`: SQL Server connection string to Identity Server storage.
- `TokenCleanUp:EnableCleanUp`: Flag indicating whether stale tokens will be automatically cleaned up from the database.
- `TokenCleanUp:CleanUpInterval`: Token clean up interval in minutes.
- `AllowedOrigins`: Enables support for Cross-Origin Resource Sharing. You can explicilty list the origins or use asterisk symbol as wildcard to allow all origins.  

### User Management

Identity Server exposes several resources under `/users` path to manage TAPI user accounts.

#### Get Users

Url Mask:

```
GET api/{apiVersion}/users
```

Example:

```
GET api/v1/users
```

Result:

```
[
    {
        "id": "73c06464-54ae-4aa7-8495-6c2733dbd394",
        "userName": "admin",
        "validFrom": "2018-10-02T14:42:15.3909783",
        "validTo": "2018-12-02T14:42:15.391184"
    },
    {
        "id": "df127e7c-4c60-4344-b065-6d947a226dc7",
        "userName": "bob@example.com",
        "validFrom": "2018-09-02T00:00:00",
        "validTo": "2018-12-02T00:00:00"
    }
]
```

#### Get User by Id

Url Mask:

```
GET api/{apiVersion}/users/{id}
 ```

Example:

```
GET api/v1/users/73c06464-54ae-4aa7-8495-6c2733dbd394
```

Result:
```
{
    "id": "73c06464-54ae-4aa7-8495-6c2733dbd394",
    "userName": "admin",
    "validFrom": "2018-10-02T14:42:15.3909783",
    "validTo": "2018-12-02T14:42:15.391184"
}
```

#### Create New User

Url Mask:

```
POST api/{apiVersion}/users
```

Example:

```
POST api/v1/users
```

Request Body:

```
{
  "userName": "test",
  "validFrom": "2018-10-02T14:42:15.3909783",
  "validTo": "2018-12-02T14:42:15.391184",
  "password": "testT1@"
}
```

Result:

```
{
  "id": "1e958756-40e7-4886-b58f-13055df8847c",
  "userName": "test",
  "validFrom": "2018-10-02T14:42:15.3909783",
  "validTo": "2018-12-02T14:42:15.391184"
}
```

#### Update Existing User

Url Mask:

```
PUT api/{apiVersion}/users/{id}
```

Example:

```
PUT api/v1/users/1e958756-40e7-4886-b58f-13055df8847c
```

Request Body:

```
{
    "id": "1e958756-40e7-4886-b58f-13055df8847c",
    "userName": "tes1t",
    "validFrom": "2018-10-02T14:42:15.3909783",
    "validTo": "2018-12-02T14:42:15.391184"
}
```

Important: You need to provide user id and username in request body.

#### Reset Password

Resetting password is similar to updating an existing user. However, in addition to user Id and username,  you need to also provide the old password for validation.

Url Mask:

```
PUT api/{apiVersion}/users/{id}
```

Example:

```
PUT api/v1/users/1e958756-40e7-4886-b58f-13055df8847c
```

Request Body:

```
{
    "id": "1e958756-40e7-4886-b58f-13055df8847c",
    "userName": "tes1t",
    "oldPassword": "testT1@",
    "password": "testT1@new"
}
```

#### Delete User

Url Mask:

```
DELETE api/{apiVersion}/users/{id}
```

Example:

```
DELETE api/v1/users/1e958756-40e7-4886-b58f-13055df8847c
```

### Client Management

Identity Server exposes a resource under `/clients` path to create new TAPI clients.

#### Create New Client

Url Mask:

```
POST api/{apiVersion}/clients
```

Example:

```
POST api/v1/clients
```

Request Body:

```
{
  "clientId": "mvc.client",
  "clientName": "MVC Client for TAPI",
  "accessTokenLifetime": 3600,
  "allowRefreshToken": true,
  "refreshTokenLifetime": 3600
}
```
Parameter explanation:

 - `clientId`: Unique client identifier (required). 
 - `clientName`: Descriptive name identify the client.
 - `accessTokenLifetime`: Determines the period of validity of the access token in seconds. Must be larger than or equal to 1. It is recommended to keep the lifetime of the access token short and use the refresh token to renew access token. Default value is 1 hour.
 - `allowRefreshToken`: Flag indicating whether to allow the client to refresh the access token without requiring username and password.
 - `refreshTokenLifetime`: Determines the period of validity of refresh token in seconds. Set this to 0 to reuse the same refresh token.
