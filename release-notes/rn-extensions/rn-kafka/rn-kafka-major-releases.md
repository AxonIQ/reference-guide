# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Kafka Extension.

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
