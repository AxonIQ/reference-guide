# Minor Releases

Any patch release made for an Axon project aims to resolve bugs.
This page provides an overview of patch releases for the Axon Kafka Extension.

## Release 4.5

### Release 4.5.2

This release only includes two commits, namely [245536f](https://github.com/AxonFramework/extension-kafka/commit/245536fa99086857ca63da752773c562af962da4) and [e8ddb9d](https://github.com/AxonFramework/extension-kafka/commit/e8ddb9dc77e1ab66c09a0a279394f9b5e331d6a1).
These commits ensure that both event publishing and consuming provide a workable `XStreamSerializer` default.
Plus, the extension logs warnings in case a users uses the defaults.

### Release 4.5.1

* Fixed a bug where the XStream was still a hard requirement [here](https://github.com/AxonFramework/extension-kafka/pull/214).

* Even though the extension only have it as a `test` dependency, this release also bumped `log4j` dependency to `2.15.0` [here](https://github.com/AxonFramework/extension-kafka/commit/6efd14c8108f8d991a8f07b3b526c0169f4d4e88).
 