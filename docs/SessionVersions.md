# ![logo](/docs/branding.bmp) Telemetry Analytics API

### Table of Contents
- [**Introduction**](/README.md)<br>
- [**Installation**](/docs/Installation.md)<br>
- [**Getting started**](/docs/GettingStarted.md)<br>
- [**Identity Server**](/docs/IdentityServer.md)<br>
- [**Authorization**](/docs/Authorization.md)<br>
- [**Querying Metadata**](/docs/Metadata.md)<br>
- [**Consuming Data**](/docs/ConsumingData.md)<br>
- [**Session Versions**](/docs/SessionVersions.md)<br>
- [**Views**](/docs/Views.md)<br>


## Session Versions

In TAP a session can be identified using various identifiers some of which are guarenteed to be unique everytime you stream a session using TAP while others aren't. Following are the identifiers you can use to identify a session.

1. Stream Id <br>
Stream Id is a GUID that is generated each time you stream a session from TAP. This is a guarenteed to be unique each time you stream the session. For example, if you are replaying the same historic session from a file multiple times, you will get a new `streamId` each time.
2. Id <br>
Session Id which is also a GUID is unique per session up to a replay. In other words, if you replay the same historic session, session id does not change between replays.
3. Identifier <br>
Identifer is simply a human-friendly name of the session. This is unique up to a replay just like the Id.

Even though the `streamId` is guarenteed to be unique, it's usefulness is limited to an external client as this is an auto-generated GUID from within TAP which doesn't say much about the session, for instance, in terms of its origin. `id` is the most meaningful identifier in terms of keeping track of a session more generally like tracing the session to an ADS (Atlas Data Server) session. However, teh fact that `id` is not unique between replays of the same session in TAP calls for a versioning system for sessions within TAP.

## Why Replay A Session?





