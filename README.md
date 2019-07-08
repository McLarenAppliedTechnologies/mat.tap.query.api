# ![logo](/docs/branding.bmp) Telemetry Analytics API

### Table of Contents
- [**Introduction**](/README.md)<br>
- [**Installation**](/docs/Installation.md)<br>
- [**Getting started**](/docs/GettingStarted.md)<br>
- [**Identity Server**](/docs/IdentityServer.md)<br>
- [**Authorization**](/docs/Authorization.md)<br>
- [**Querying Metadata**](/docs/Metadata.md)<br>
- [**Session Versions**](/docs/SessionVersions.md)<br>
- [**Consuming Data**](/docs/ConsumingData.md)<br>
- [**Views**](/docs/Views.md)<br>


# Introduction

TAPI stands for Telemetry Analytics API. It allows platform-independent access to historic data. It provides an abstraction between specific storage technology used such as SqlRace and InfluxDb while adding analytics capabilities like data masking.

One instance of TAPI aggregates multiple storage instances. REST API provides access to session metadata, session telemetry data but also results of live models.

![](/docs/TapiDiagram.png)


