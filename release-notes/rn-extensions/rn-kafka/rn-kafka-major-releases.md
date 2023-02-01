# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Kafka Extension.

## Release 4.7.0

### Enhancements

- Add Spring Boot 3 autoconfiguration support. [#378](https://github.com/AxonFramework/extension-kafka/pull/378)

### Bug Fixes

- Fix memory leak [#365](https://github.com/AxonFramework/extension-kafka/pull/365)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@gklijs](https://github.com/gklijs)

## Release 4.6.0

If you're curious about the dependency upgrades made in this release we refer to [this](https://github.com/AxonFramework/extension-kafka/releases/tag/axon-kafka-4.6.0) page.

### Features

- Adds an optional to use Kafka based token store which can be used to … [#281](https://github.com/AxonFramework/extension-kafka/pull/281)
- Fix issue 260 by adding a topic function. [#264](https://github.com/AxonFramework/extension-kafka/pull/264)
- Create a TokenStore based on a kafka topic. [#261](https://github.com/AxonFramework/extension-kafka/issues/261)
- Allow filtering or sending different event to different topics in the publisher. [#260](https://github.com/AxonFramework/extension-kafka/issues/260)

### Enhancements

- Add integration test for the source configurer [#324](https://github.com/AxonFramework/extension-kafka/pull/324)
- Add Cloud event serializer [#321](https://github.com/AxonFramework/extension-kafka/pull/321)
- KafkaPublisher should not be needed when you only want a fetcher. [#306](https://github.com/AxonFramework/extension-kafka/issues/306)
- Fix a code smell I created earlier. [#298](https://github.com/AxonFramework/extension-kafka/pull/298)
- Prepare for 4.6.0 release, also stop creating and deleting the same topics, which makes the tests more reliable. [#293](https://github.com/AxonFramework/extension-kafka/pull/293)
- Updated serializer configuration and made them lazy [#280](https://github.com/AxonFramework/extension-kafka/pull/280)
- Fix sonar issues, either by improving the code, or by suppressing the warning. [#256](https://github.com/AxonFramework/extension-kafka/pull/256)
- Go through Sonar issues and either improve the code, or ignore the issue. [#254](https://github.com/AxonFramework/extension-kafka/issues/254)
- Clean up the pom's, move version numbers to the properties section [#253](https://github.com/AxonFramework/extension-kafka/pull/253)
- Removed beta note from README.md, and added myself as reviewer. [#250](https://github.com/AxonFramework/extension-kafka/pull/250)
- Remove beta stage note in the readme. [#249](https://github.com/AxonFramework/extension-kafka/issues/249)
- chore: fix the build problems [#201](https://github.com/AxonFramework/extension-kafka/pull/201)
- Cleanup Maven POMs [#200](https://github.com/AxonFramework/extension-kafka/issues/200)
- Separate execution of unit tests from integration tests [#199](https://github.com/AxonFramework/extension-kafka/issues/199)
- feature: implement support for event upcasters, fix #193 [#195](https://github.com/AxonFramework/extension-kafka/pull/195)
- Support Event Upcasters during reading of events [#193](https://github.com/AxonFramework/extension-kafka/issues/193)

### Bug Fixes

- Remove KafkaPublisher as conditional, such that it's easier to use the autoconfig. [#307](https://github.com/AxonFramework/extension-kafka/pull/307)

### Dependency Upgrade

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@gklijs](https://github.com/gklijs)
- [@zambrovski](https://github.com/zambrovski)
- [@lfgcampos](https://github.com/lfgcampos)


## Release 4.5

Release 4.5 marks the point where the Kafka Extension moved away from the release candidate state.
We marked this effort under issue [#167](https://github.com/AxonFramework/extension-kafka/pull/167), which upgraded this project to Axon Framework 4.5.

Before the upgrade, we validated whether the last remaining outstanding issues required work on our end.
None of them imposed changes on the extension side but instead warranted adjustments for the user.
As such, the change list for this release is meager but does conclude the 'release candidate state.'

## Release 4.0

Release 4.0 has seen several release candidates:

## Release 4.0 - Release Candidate 3

We solidified the API of the Kafka extension with the following main points:

* The original implementation only allowed users to use this extension as a streamable message source.
  However, as Kafka provides a lot of internal logic to segment, start and stop a stream, making it a subscribing solution is feasible.
  Issue [#17](https://github.com/AxonFramework/extension-kafka/issues/17) thus introduces a `SubscribingKafkaMessageSource`.
  This adjustment makes the Kafka extension a viable solution for the `TrackingEventProcessor` *and* the `SubscribingEventProcessor`.
  Furthermore, it allows the user to choose Axon's logic of partitioning or Kafka's.

* Contributor `zambrovski` did a tremendous job enhancing this extension's configuration in pull request [#11](https://github.com/AxonFramework/extension-kafka/pull/11).
  On top of that, he included a sample application showing how you can use the Kafka extension.

* When using this extension as a streaming source in Axon, it automatically constructed `KafkaTrackingTokens`.
  These tokens were, however, unaware of the topic they stored the progress for.
  Issue [#28](https://github.com/AxonFramework/extension-kafka/issues/28) addressed the problem, which we resolved after this in pull request [#29](https://github.com/AxonFramework/extension-kafka/pull/29).

For a complete list of all the adjustments, we refer to the [release notes](https://github.com/AxonFramework/extension-kafka/releases/tag/axon-kafka-4.0).

Note that this is still a _release candidate_.
As such, users should consider we might introduce API changes in future releases.

## Release 4.0 - Release Candidate 2

We introduced several minor API changes in this version.
We released it to provide users a window of opportunity to further verify the current implementation

Note that this is still a _release candidate_.
As such, users should consider we might introduce API changes in future releases.

## Release 4.0 - Release Candidate 1

We split off the Kafka logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.

Note that we adjusted the Kafka package into a _release candidate_.
As such, users should consider we might introduce API changes in future releases.
