# Bug Fixes

Any patch release made for an Axon project is tailored towards resolving bugs. This page aims to provide a dedicated overview of patch releases per project.

## Axon Framework

### Release 4.3

* A `ConcurrencyException` was thrown when an aggregate was created at two distinct JVM's at the same time.

  As `ConcurrencyException`s are typically retryable,

   the creation command would be issued again if a `RetryScheduler` was in place.

  Retrying this operation is however useless and hence has been replaced for an `AggregateStreamCreationException` in pull request [\#1333](https://github.com/AxonFramework/AxonFramework/pull/1333). 

* The test fixtures for state-stored aggregates did unintentionally not allow resource injection.

  This problem has been resolved in pull request [\#1315](https://github.com/AxonFramework/AxonFramework/pull/1315).

* The `MultiStreamableMessageSource` did not deal well with one or several exceptional streams.

  Hence exception handling has been improved on this matter in [\#1325](https://github.com/AxonFramework/AxonFramework/pull/1325).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.3%22++label%3A%22Type%3A+Bug%22).

### Release 4.2.2

* In a distributed setup, the `DisruptorCommandBus` was not always correctly identified as being the local segment.

  Due to this, aggregate repositories weren't created by the `DisruptorCommandBus` as is required in such a configuration.

  This was marked in [\#874](https://github.com/AxonFramework/AxonFramework/issues/874) and resolved through [\#1287](https://github.com/AxonFramework/AxonFramework/pull/1287).

* As described in [\#1274](https://github.com/AxonFramework/AxonFramework/issues/1274),

   a query handler with return type `Future` was not being returned at all but threw an exception.

  Pull request [\#1323](https://github.com/AxonFramework/AxonFramework/pull/1323) solved that in 4.2.2.

* An issue was solved where the `JdbcAutoConfiguration` unintentionally depended on a JPA specific class.

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.2.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.2.1

* A one-to-many `Upcaster` instance tied to Axon Server would only use the first event result and ignore the rest.

  This issue has been resolved in pull request [\#1264](https://github.com/AxonFramework/AxonFramework/pull/1264).

* The `axon-legacy` module's `GapAwareTrackingToken` did not implement the `TrackingToken` interface. 

  This was marked in issue [\#1230](https://github.com/AxonFramework/AxonFramework/issues/1230) and resolved in [\#1231](https://github.com/AxonFramework/AxonFramework/pull/1231).

* The builders of the `ExponentialBackOffIntervalRetryScheduler` and `IntervalRetryScheduler` previously

   did not implement the `validate()` method correctly.

  Through this a `NullPointerException` could occur on start-up,

   as marked in [\#1293](https://github.com/AxonFramework/AxonFramework/issues/1293).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.2.1%22++label%3A%22Type%3A+Bug%22).

### Release 4.2

* An Aggregate's `Snapshotter` was not auto configured when Spring Boot is being used, as was filed under [\#932](https://github.com/AxonFramework/AxonFramework/issues/932).
* The `CommandResultMessage` was returned as `null` when using the [`DisruptorCommandBus`]().

  This was solved in pull request [\#1169](https://github.com/AxonFramework/AxonFramework/pull/1169).

* The `ScopeDescriptor` used by the `DeadlineManager` had serialization issue when a user would migrate from an Axon 3.x application to Axon 4.x.

  The `axon-legacy` package has been expanded to contain legacy `ScopeDescriptor`s to resolve this problem.

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.1.2

* A dependency on `XStream` was enforced undesirably through the Builder pattern introduced in 4.0. 

  This has been resolved by using a `Supplier` of a `Serializer` in the Builders instead, as described under [this](https://github.com/AxonFramework/AxonFramework/issues/1054) issue.

* Due to a hierarchy issue in the Spring Boot auto configuration, the `JdbcTokenStore` was not always used as expected.

  The ordering has been fixed under issue [\#1077](https://github.com/AxonFramework/AxonFramework/issues/1077).

* The ordering of message handling functions was incorrect according to the documentation.

  Classes take precedence over interface, and the depth of interface hierarchy is calculated based on the inheritance level \(as described [here](https://github.com/AxonFramework/AxonFramework/pull/1129)\).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.1.1

* Query Dispatch Interceptors were no called correctly when a [subscription query](../axon-application-development/query-handling/dispatching-queries.md#subscription-queries) was performed when Axon Server was used as the `QueryBus`.

  This issue was marked [here](https://github.com/AxonFramework/AxonFramework/issues/1013) and resolved in pull request [\#1042](https://github.com/AxonFramework/AxonFramework/pull/1042).

* When Axon Server was \(auto\) configured without being able to connect to an actual instance, processing instructions were incorrectly dispatched regardless.

  Pull request [\#1040](https://github.com/AxonFramework/AxonFramework/pull/1040) resolves this by making sure an active connection is present.

* The Spring Boot auto configuration did not allow the exclusion of the `axon-server-connector` dependency due to a direct dependency on classes.

  This has been resolved by expecting fully qualified class names as Strings instead \(resolved under [this](https://github.com/AxonFramework/AxonFramework/pull/1041) pull request\).

* The `JpaEventStorageEngine` was not wrapping the `appendEvents` operation in a transaction.

  Problem has been resolved under issue [\#1035](https://github.com/AxonFramework/AxonFramework/issues/1035). 

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1.1%22++label%3A%22Type%3A+Bug%22).

### Release 4.1

* Command an Query message priority was not set correctly for the Axon Server Connector.

  This issue has been addressed under bug [\#1004](https://github.com/AxonFramework/AxonFramework/pull/1004).

* The `CapacityMonitor` was not registered correctly for Event Processor, which user "Sabartius" resolved under issue [\#961](https://github.com/AxonFramework/AxonFramework/issues/961).
* Some exception were not reported correctly and/or clearly when utilizing the Axon Server Connector \(issue marked under number [\#945](https://github.com/AxonFramework/AxonFramework/pull/945)\).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.4

* Deserialization failures were accidentally swallowed by the command and query gateway \(marked under [\#967](https://github.com/AxonFramework/AxonFramework/issues/967)\).
* Resolved an issue where custom exception in a Command Handling constructor caused `NullPointerExceptions`.

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.4%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.3

* The `SimpleQueryBus` reported exceptions on the initial result incorrectly upon performing a subscription query.

  Issue has been described and resolved under [\#913](https://github.com/AxonFramework/AxonFramework/issues/913). 

* Resolved issue where the the "download Axon Server" message was shown upon a reconnect of an application to a Axon Server node.
* Large global index gaps between events caused issues when querying the event stream \(described [here](https://github.com/AxonFramework/AxonFramework/issues/419)\).
* Fixed inconsistency in the `GlobalSequenceTrackingToken#covers(TrackingToken)` method.   

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.3%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.2

* A timeout was thrown instead of a exception by Axon Server when a duplicate aggregate id was created, which is resolved in [\#903](https://github.com/AxonFramework/AxonFramework/issues/903). 
* Command or Query handling exception were no properly serialized through Axon Server \(resolved in [\#904](https://github.com/AxonFramework/AxonFramework/pull/904)\). 

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.1

* Resolved `QueryUpdateEmitter` configuration for the Axon Server connector set up \(see issue [here](https://github.com/AxonFramework/AxonFramework/issues/896)\).
* For migration purposes legacy `TrackingTokens` should have been added, which is resolved [here](https://github.com/AxonFramework/AxonFramework/issues/886).
* Event Processing was stopped after a reconnection with Axon Server. Resolve the problem in issue [\#883](https://github.com/AxonFramework/AxonFramework/issues/883). 

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.1%22++label%3A%22Type%3A+Bug%22).

### Release 4.0

The bugs marked for release 4.0 were issues introduced to new features or enhancements. As such they should not have impacted users in any way. Regardless, the full list can be found [here](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0%22++label%3A%22Type%3A+Bug%22).

## Axon Framework Extensions

### JGroups 4.2

As marked under [this](https://github.com/AxonFramework/extension-jgroups/issues/4) issue, the command callback will not be called if the connection between JGroups peers dies whilst a command is in transit. Credits go to "sgrimm-sg" for filing the issue and solving [it](https://github.com/AxonFramework/extension-jgroups/pull/5).

### JGroups 4.1

If cluster connection message came in quickly after starting the connection, a `NullPointerException` could be thrown. This issue was resolved for release 4.1 [here](https://github.com/AxonFramework/extension-jgroups/issues/1).

### Mongo 4.0.1

The Mongo extension incorrectly used the content type instead of the tokens type upon storing a serialized token. The issue was marked and resolved under [\#1](https://github.com/AxonFramework/extension-mongo/issues/1).

### Spring Cloud 4.2

When using the [Kubernetes](https://spring.io/projects/spring-cloud-kubernetes) implementation of Spring Cloud the `SpringCloudCommandRouter` would throw `NullPointerException`s. This occurs because Spring Cloud Kubernetes does not support the `ServiceInstance`'s meta data field, which the `SpringCloudCommandRouter` relies on. [This](https://github.com/AxonFramework/extension-springcloud/pull/10) pull request introduced a null check to ensure the null pointer would not be thrown again.

### Spring Cloud 4.1

The `SpringCloudCommandRouter` failed to correctly connect to a Spring Cloud Discovery Service if the node did not contain any Command Handler methods. This undesired behaviour was marked by user "travikk" and made more lenient under [this](https://github.com/AxonFramework/extension-springcloud/issues/1).

