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

# Identity Server

Identity Server implements OpenID Connect and provides authentication services for Telemetry Analytics API. Identity Server provides following services.

- Provides a REST API to manage TAPI users;
- Issues access tokens to authorize access to TAPI resources.

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
sudo systemctl stop MAT.TAP.IdentityServer.Writer 
sudo systemctl start MAT.TAP.IdentityServer.Writer 
```

or configure your config in /opt/MAT.TAP.IdentityServer/appConfiguration.Production.json

#### Basic usage

In order to use IdentityServer, add the relevant configuration in `appConfiguration.Production.json` file and start service using

    dotnet MAT.TAP.AAS.IdentityServer.dll --urls="http://*:5000"

A sample configuration and an explanation of settings is given below.

```
{
  "ConnectionStrings": {
    "IdentityDatabase": "server=.\\SQLEXPRESS;Initial Catalog=MAT-TAP-IdentityServer;User Id=test;Password=test;"
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
  "OAuthServer": "http://localhost:5000"
}
```

- `OAuthServer`: Address of the OAuthServer for authorization (User CRUD API). **If you are accessing API from outside using external IP address you might need to put external address here.**
- `InitializeDatabase`: True to initialize database configured in ConnectionStrings section.
- `ConnectionStrings`: SQL Server connection string to Identity server storage.

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

Resetting password is similar to updating an existing user. However, in addition to user Id and username,  you need to also provide old password for validation.

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
