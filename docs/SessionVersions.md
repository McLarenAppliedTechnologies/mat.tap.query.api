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

#### Why replay a session?

The same historic session may be replayed in TAP for various reasons. Some of the more common uses for replaying the same session are:

1. Play a down-sampled version of the original telemetry session;
2. Run data science models which may be versioned themselves on the same telemetry session.

In order to support the above scenarios, we have introduced session versioning in TAP which allows you to group and version sessions along with metadata about any custom analytics models you may have used during a session replay.

The json representation of a session model:

```
{
    "id": "01bdac9f-18d9-4e8c-956b-c397206bf5a4",
    "streamId": "08fb78fa-6547-4a60-8a58-f3d6c1dd2978",
    "identifier": "Identifier5",
    "timeOfRecording": "2018-11-29T00:00:00Z",
    "sessionType": "StreamingSession",
    "start": "2019-03-28T14:36:42.1170779Z",
    "end": "2019-03-28T14:36:42.1170779Z",
    "lapsCount": 10,
    "state": "Closed",
    "topicName": "TopicName5",
    "group": "Group10",
    "version": 1,
    "configuration": "{ \"key\": \"value\" }",
    "sessionDetails": []
  }
```

In the above json, `group`, `version` and `configuration` carry information about a session version. `group` indicates any group this session model belongs to. `version` indicates the session version which you are free to use the way best fit your needs (e.g. version the session together with the data science model, etc.). You can use `configuration` property store detailed information about the session version (e.g. a stringified representation of the data science model such as a json).

#### Querying session versions





