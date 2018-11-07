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

# Installation
## SqlRace

### Prerequisites:
- SQLRace database installed
- SQLRace license installed on machine

Run msi installer and follow instructions. **Note that API will clash with Atlas 10 and SQLRace so it has to be deployed to a standalone machine.**

## InfluxDb
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
One the examples how to run TAPI is systemd daemon service. In the release bundle, there is a shell script **daemon_deploy.sh** for daemon installation. 

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
journalctl --unit MAT.TAP.TelemetryAnalytics.API.service --follow -n 100
```

or start and stop by 

```
sudo systemctl stop MAT.TAP.TelemetryAnalytics.API
sudo systemctl start MAT.TAP.TelemetryAnalytics.API
```

or configure your config in /opt/MAT.TAP.TelemetryAnalytics.API/appConfiguration.Production.jso

#### Basic usage

In order to use IdentityServer, add the relevant configuration in `appConfiguration.Production.json` file and start service using

    dotnet MAT.TAP.AAS.TelemetryAnalytics.API.dll --urls="http://*:5000"

A sample configuration and an explanation of settings is given below.

```
{
  "ConnectionStrings": {
    "Model": "server=.\\SQLEXPRESS;Initial Catalog=Telemetry.Analytics.API.Influx;User Id=test;Password=test;",
  },
  "InitializeDatabase": true,
  "OAuthServer": "http://localhost:5000"
}
```

- `OAuthServer`: Address of the OAuthServer for authorization (User CRUD API). **If you accessing API from outside using external IP adress you might need to put external address here.**
- `InitializeDatabase`: True to initialize database configured in ConnectionStrings section.
- `ConnectionStrings`: SQL Server connection string to Identity server storage.
