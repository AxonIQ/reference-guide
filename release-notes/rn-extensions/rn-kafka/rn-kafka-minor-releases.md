# Minor Releases

Any patch release made for an Axon project aims to resolve bugs.
This page provides an overview of patch releases for the Axon Kafka Extension.

## Release 4.5

### Release 4.5.4

* Whenever the `StreamableKafkaMessageSource` started, it created a new consumer group.
  This implementation will eventually construct many groups within Kafka that are never used, as contributor `aupodogov` noted in [this](https://github.com/AxonFramework/extension-kafka/issues/273) issue.
  We solved this issue by manually validating all the partitions instead of constructing a new consumer group each time.
  For those interested, the pull request for this can be found [here](https://github.com/AxonFramework/extension-kafka/pull/284).
* Contributor `zambrovski` noticed that when `FetchEventTask` dies, it did not put the Event Processor in error mode even though it should.
  To solve this, we ensured the error was rethrown far enough for the Event Processor to react to it accordingly.
  If you're curious about the solution, you can find the changes in pull request [#287](https://github.com/AxonFramework/extension-kafka/pull/287).

You can find the complete list of changes in release 4.5.4 [here](https://github.com/AxonFramework/extension-kafka/releases).

### Release 4.5.3

The 4.5.3 release 're-enables' [Spring Cloud Developer Tools](https://docs.spring.io/spring-boot/docs/1.5.16.RELEASE/reference/html/using-boot-devtools.html) for the Kafka Extension.
The support for this was accidentally removed, marked contributor `tiger-seo` in issue [#240](https://github.com/AxonFramework/extension-kafka/issues/240).
Commit [9bc2345](https://github.com/AxonFramework/extension-kafka/commit/9bc2345692f445e1ee2575c601956078d06946df) reinstantes this behavior. 

### Release 4.5.2

This release only includes two commits, namely [245536f](https://github.com/AxonFramework/extension-kafka/commit/245536fa99086857ca63da752773c562af962da4) and [e8ddb9d](https://github.com/AxonFramework/extension-kafka/commit/e8ddb9dc77e1ab66c09a0a279394f9b5e331d6a1).
These commits ensure that both event publishing and consuming provide a workable `XStreamSerializer` default.
Plus, the extension logs warnings in case a users uses the defaults.

### Release 4.5.1

* Fixed a bug where the XStream was still a hard requirement [here](https://github.com/AxonFramework/extension-kafka/pull/214).

* Even though the extension only have it as a `test` dependency, this release also bumped `log4j` dependency to `2.15.0` [here](https://github.com/AxonFramework/extension-kafka/commit/6efd14c8108f8d991a8f07b3b526c0169f4d4e88).
 