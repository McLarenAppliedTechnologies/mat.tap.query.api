# ![logo](/docs/branding.bmp) Telemetry Analytics API

![Build Status](https://mat-ocs.visualstudio.com/Telemetry%20Analytics%20Platform/_apis/build/status/MAT.TAP.TelemetryAnalytics.API/MAT.TAP.TelemetryAnalytics.API%20-%20Pull%20Request%20Gateway?branchName=develop)

### Table of Contents
- [**Introduction**](/README.md)<br>
- [**Getting started**](/docs/GettingStarted.md)<br>
- [**Authorization**](/docs/Authorization.md)<br>
- [**Querying Metadata**](/docs/Metadata.md)<br>
- [**Consuming Data**](/docs/ConsumingData.md)<br>
- [**Views**](/docs/Views.md)<br>

# Saving Data

Telemetry data is saved to various storage implementations such as SqlRace and InfluxDb. This article discusses saving telemetry data to InfluxDb as time-series data and metadata to MSSQL Server.

## InfluxDb Writer

InfluxDb Writer consists of two components: [Influx Connector](#influxdb-connector) and [Influx Writer](#influxdb-writer-1).

### InfluxDb Connector

Influx Connector is a .NET Standard compliant library providing data access services for session metadata and telemetry data storage. Session metadata DAL services work with any relational database provider while telemetry data services are defined for InfluxDb. Since it's .NET Standard compliant, Influx Connector can be used by any implementation of .NET or .NET Core and is platform independent.

In order to use Influx Connector, you need to have command-line tools for EF Core installed:

- Install [.NET Core SDK](https://www.microsoft.com/net/download).
- Install EF Core design-time tools by running the following command on Command-line Interface or Terminal.

    ```
    dotnet add package Microsoft.EntityFrameworkCore.Design
    ```

For more information on setting up EF Core CLI tools refer to [official documentation](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/#install-the-tools).

All the commands below must be run on Command-line Interface, PowerShell or Terminal depending on the platform.

#### Setting up database for metadata

Specify the database connection string in `config.json` found in the Influx Connector root directory. For SQL Server:

    {
        "ConnectionStrings": {
            "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Initial Catalog=[DatabaseName];User Id=[Username];Password=[Password];"
        }
    }

Relational database provider can be replaced by specifying the appropriate middleware in `SessionsDbContextFactory` class. Default implementation uses MSSQL Server.

Run the following command to create the database and the schema:

    dotnet ef database update

#### Customizing metadata model

Modify the `SessionsDbContext` class and create the migrations:

    dotnet ef migrations add [MigrationName]

Apply migrations to the database:

    dotnet ef database update

For a more comprehensive guide on managing migrations, please refer to [EF Core Migrations](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/).

### InfluxDb Writer

InfluxDb writer subscribes to message brokers and saves telemetry data and session metadata in real-time to time-series and relational databases respectively. InfluxDb Writer is platform-independent and hence can be deployed on Windows or Unix-based systems as a service.

#### Basic usage

In order to use InfluxDb writer, add the relevant configuration in `config.json` file and start service using

    dotnet MAT.TAP.AAS.InfluxDb.Writer.dll

A sample configuration and an explanation of settings is given below.

    {
        "BrokerList": "xx.xxx.x.xx",
        "DependencyUrl": "http://[hostname/ip_address]/api/dependencies/",
        "DependencyGroup": "[dependency group identifier]",
        "BatchSize": 100000,
        "ThreadCount": 5,
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
- `Connections`: Contains all the database connection information organized by the topic (e.g. Kafka topics.)
- `[TopicName]` : Change the value here depending on the message queue topic you want to subscribe to.
- `InfluxConnections`: Contains all the InfluxDb connection strings. Influx writer supports multiple InfluxDb connections per topic under [labels](#label-supprt). If you plan to use just one InfluxDb instance, use asterisk symbol (*) as a wildcard key. This means, all telemetry data under the topic will be saved to InfluxDb specified in `InfluxDbUrl` regardless of the label.
- `SqlServerConnectionString`: Connection string for session metadata relational database. Influx Writer supports one metadata connection per topic.

#### Label Supprt

Labels are a concept introduced in Influx Writer to provide support for scaling and High Availability for Influx Writer and the time-series database in a flexible manner. Using labels, you can partition a topic and save data for a single topic in multiple InfluxDb instances using multiple instances of Influx Writers.

An example usage of labels in F1 Racing scenario is to give separate labels for each driver and specify `InfluxDbUrl` per label. This way, you can deploy and manage separate instances of InfluxDb Writers and databases per driver in which each instance will only have to handle data related to the specific label, hence reducing the load on each instance.

#### Logging

Influx Writer has extensive logging and uses Nlog for logger implementation. You can configure logging in the `nlog.config` file or provide your own logging configuration. More information on available configuration options can be found [here](https://github.com/nlog/nlog/wiki/Configuration-file).