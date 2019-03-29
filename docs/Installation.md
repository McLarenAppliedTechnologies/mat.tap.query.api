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

# Installation
## SqlRace

### Prerequisites:
- SQLRace database installed
- SQLRace license installed on machine

Run msi installer and follow instructions. **Note that API will clash with Atlas 10 and SQLRace so it has to be deployed to a standalone machine.**

## InfluxDb
### Deployment

Install [**Identity Server**](/docs/IdentityServer.md) before installing MAT.TAP.TelemetryAnalytics.API.

#### .NET Core runtime
First you need to install .NET Core 2.1 runtime. You can download it [here](https://www.microsoft.com/net/download/dotnet-core/2.1). Following are the installation steps for Ubuntu 18.04 LTE:

```
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt-get --yes install apt-transport-https
sudo apt-get update
sudo apt-get --yes install aspnetcore-runtime-2.1
```

#### Daemon installation
One way to run TAPI is as a systemd daemon service. In the release bundle, there is a shell script **daemon_deploy.sh** for daemon installation. 

Before you run it, execute following commands:
```
awk 'BEGIN{RS="^$";ORS="";getline;gsub("\r","");print>ARGV[1]}' daemon_deploy.sh
sudo chmod 777 daemon_deploy.sh
```

Then run:
```
./daemon_deploy.sh
```

You can verify it with:

```
journalctl --unit MAT.TAP.TelemetryAnalytics.API.service --follow -n 100
```

You can use the following commands to stop or start the service. 

```
sudo systemctl stop MAT.TAP.TelemetryAnalytics.API.service
sudo systemctl start MAT.TAP.TelemetryAnalytics.API.service
```

TAPI uses `/opt/MAT.TAP.TelemetryAnalytics.API/appsettings.Production.json` as configuraiton file. You can modify configuration by changing the values in this file.

#### Basic usage

In order to use Identity Server, add the relevant configuration in `appsettings.Production.json` file and start service using

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

Note that you can run Identity Server as a service just like TAPI using the **daemon_deploy.sh** file in the Identity Server release bundle. This is the recommended deployment for a production environment.

**Please note, that daemon is using configuration file from /opt/MAT.TAP.IdentityServer/appsettings.Production.json and that service needs to restart after making any changes to the configuration file.**

- `OAuthServer`: Address of the OAuthServer for authorization (User CRUD API). **If you are accessing the API from outside using an external IP adress you may need to use that external address here.**
- `InitializeDatabase`: Set `True` to initialize database configured in ConnectionStrings section. This creates the database if it doesn't exist and applies any pending database migrations.
- `ConnectionStrings`: SQL Server connection string to Identity Server storage.
