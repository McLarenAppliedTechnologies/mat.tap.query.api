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

    {
        "BrokerList": "xx.xxx.x.xx",
        "DependencyUrl": "http://[hostname/ip_address]/api/dependencies/",
        "DependencyGroup": "[dependency group identifier]",
        "BatchSize": 100000,
        "ThreadCount": 5,
        "InitializeDatabase": true,
        "Connections": {
            "[TopicName]": {
            "InfluxConnections": {
                "*": {
                "InfluxDbUrl": "http://[hostname/ip_address]"
                }
            },
            "SqlServerConnectionString": "Server=(localdb)\\MSSQLLocalDB;Initial Catalog=[DatabaseName];User Id=[Username];Password=[Password];"
            }
        }
    }

- `BrokerList`: Address of the message broker cluster.
- `DependencyUrl` and `DependencyGroup`: Settings related to ATLAS configuration and session metadata.
- `BatchSize`: Number of telemetry samples to be saved to InfluxDb at a time.
- `ThreadCount`: Number of processor threads to be used by the Influx Writer. A value larger than 1 can improve throughput of the writer in a machine that supports multithreading.
- `InitializeDatabase`: True to initialize databases configured in connections section.
- `Connections`: Contains all the database connection information organized by the topic (e.g. Kafka topics.)
- `[TopicName]` : Change the value here depending on the message queue topic you want to subscribe to.
- `InfluxConnections`: Contains all the InfluxDb connection strings. Influx writer supports multiple InfluxDb connections per topic under [labels](#label-supprt). If you plan to use just one InfluxDb instance, use asterisk symbol (*) as a wildcard key. This means, all telemetry data under the topic will be saved to InfluxDb specified in `InfluxDbUrl` regardless of the label.
- `SqlServerConnectionString`: Connection string for session metadata relational database. Influx Writer supports one metadata connection per topic.