# ![logo](/docs/branding.bmp) Telemetry Analytics API

![Build Status](https://mat-ocs.visualstudio.com/Telemetry%20Analytics%20Platform/_apis/build/status/MAT.TAP.TelemetryAnalytics.API/MAT.TAP.TelemetryAnalytics.API%20-%20Pull%20Request%20Gateway?branchName=develop)

### Table of Contents
- [**Introduction**](/README.md)<br>
- [**Getting started**](/docs/GettingStarted.md)<br>
- [**Authorization**](/docs/Authorization.md)<br>
- [**Querying Metadata**](/docs/Metadata.md)<br>
- [**Consuming Data**](/docs/ConsumingData.md)<br>
- [**Views**](/docs/Views.md)<br>

# Identity server

TODO: some generic description

### Deployment
#### .NET Core runtime
First you need to install .NET Core 2.1 runtime. You can donwload it [here](https://www.microsoft.com/net/download/dotnet-core/2.1). Example for Ubuntu 18.04 LTE: 

```
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt-get --yes install apt-transport-https
sudo apt-get update
sudo apt-get --yes install aspnetcore-runtime-2.1
```

#### Daemon installation
One the examples how to run Identity server is systemd daemon service. In the release bundle, there is a shell script **daemon_deploy.sh** for daemon installation. 

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

or configure your config in /opt/MAT.TAP.IdentityServer/config.json

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

- `OAuthServer`: Address of the OAuthServer for authorization (User CRUD API). **If you accessing API from outside using external IP adress you might need to put external address here.**
- `InitializeDatabase`: True to initialize database configured in ConnectionStrings section.
- `ConnectionStrings`: SQL Server connection string to Identity server storage.
