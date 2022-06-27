# Minor Releases

Any patch release made for an Axon project is tailored towards resolving bugs. This page aims to provide a dedicated overview of patch releases per project.

## Release 4.5

### Release 4.5.12

#### Bug Fixes

- Ensure commands and queries are processed in FIFO order [#2263](https://github.com/AxonFramework/AxonFramework/pull/2263)
- Commands with same priority are not handled in the correct order [#2257](https://github.com/AxonFramework/AxonFramework/issues/2257)

### Release 4.5.11

#### Enhancements

- Release announcement on discuss [#2256](https://github.com/AxonFramework/AxonFramework/pull/2256)

#### Bug Fixes

- [#2242] Correctly support null-identifier and no-event scenarios from Command Handling constructors, `Always`, and `Create-If-Missing` creation policies [#2248](https://github.com/AxonFramework/AxonFramework/pull/2248)
- Check attribute filter deep equals [#2246](https://github.com/AxonFramework/AxonFramework/pull/2246)
- Fix Duplicate command handler resolver is triggered in polymorphic ag… [#2245](https://github.com/AxonFramework/AxonFramework/pull/2245)
- Duplicate command handler resolver is triggered in polymorphic aggregates [#2243](https://github.com/AxonFramework/AxonFramework/issues/2243)
- AggregateTestFixture throws AggregateNotFoundException when a command handler with a creation policy applies no events [#2242](https://github.com/AxonFramework/AxonFramework/issues/2242)

#### Dependency Upgrade

- Bump spring-framework-bom from 5.3.20 to 5.3.21 [#2255](https://github.com/AxonFramework/AxonFramework/pull/2255)
- Bump metrics-core from 4.1.31 to 4.1.32 [#2247](https://github.com/AxonFramework/AxonFramework/pull/2247)

### Release 4.5.10

* Axon's test fixtures perform a "deep equals" operation, using reflection as they go. JDK17,
   rightfully so, does not allow that for all classes. 
  To solve scenarios where users utilize objects from, for example, `java.lang`,
   we have introduced a distinct `DeepEqualsMatcher` in pull request [#2210](https://github.com/AxonFramework/AxonFramework/pull/2210). 
  This matcher implementation considers the situation that an `InaccessibleObjectException` might be thrown from Axon's test fixtures,
   correctly dealing with the scenario by assuming the assertion failed.
* Contributor [`fabio-couto`](https://github.com/fabio-couto) noticed a predicament within the `PooledStreamingEventProcessor` (PSEP for short) when they were facing connectivity issues with their RDBMS. 
  In the face of these issues, the PSEP coordinator is incapable of fetching events, resulting in canceled work packages. 
  As part of canceling, the PSEP actively tries to release token claims, which is yet another database operation. 
  This loop of several connectivity issues causes the PSEP to enter a state it could not recover from. 
  Pull request [#2225](https://github.com/AxonFramework/AxonFramework/pull/2225), provided by `fabio-couto`, solves this predicament.
* A fix was introduced to the `EventTypeUpcaster` to solve issues further down the upcasting chain. 
  Contributor [`dakr0013`](https://github.com/dakr0013) noted that upcaster invoked _after_ an `EventTypeUpcaster` failed because the expected intermediate event type was adjusted to `Object`.
  `dakr0013` provided a pull request, which we made some adjustments in PR [#2177](https://github.com/AxonFramework/AxonFramework/pull/2177) to accommodate additional scenarios.

You can check out the [release notes](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.10) when you're looking for an exhaustive list of all the changes.

### Release 4.5.9

This release brings three adjustments worth mentioning to the framework, namely:

1. Contributor `oysteing` opened issue [#2154](https://github.com/AxonFramework/AxonFramework/issues/2154),
    describing that the `ReplayStatus` enumeration never entered the `REPLAY` status for a `PooledStreamingEventProcessor`. 
   We resolved this finding in pull request [#2168](https://github.com/AxonFramework/AxonFramework/pull/2168) by ensuring the `TrackingToken` carries the replay status as intended.
2. The `AggregateTestFixture` incorrectly assumed a test succeeded in the absence of an exception when you would use the `expectExceptionMessage` validation step. 
   We resolved this predicament in pull request [#2127](https://github.com/AxonFramework/AxonFramework/pull/2127).
3. Lastly, we further upgraded the XStream dependency for a CVE in [this](https://github.com/AxonFramework/AxonFramework/pull/2097) pull request. 
   This time, for [CVE-2021-43859](https://x-stream.github.io/CVE-2021-43859.html).

For an exhaustive list of the changes in 4.5.9, we refer to the [release notes](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.9).

### Release 4.5.8

This release brings two adjustments worth mentioning to the framework.
Namely:

1. We spotted a bug within the `PooledStreamingEventProcessor` (PSEP).
   More specifically, whenever a subset of the tokens for the PSEP existed, calculating the lower bound of a token would cause failures.
   We addressed this predicament in pull request [#2082](https://github.com/AxonFramework/AxonFramework/pull/2082).
2. We introduce an enhancement in the API of the `CommandGateway`.
   You can now directly insert `MetaData` whenever using the `CommandGateway#send` or `CommandGateway#sendAndWait` operations.
   You can verify the changes [#here](https://github.com/AxonFramework/AxonFramework/pull/2085).

### Release 4.5.7

This [release](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.7) contains a single fix.
Namely, pull request [#2067](https://github.com/AxonFramework/AxonFramework/pull/2067).
This pull request solves a bug that had the `PooledStreamingEventProcessor` not handle new events resulting from an `EventMultiUpcaster`.
The kudos for spotting the bug go to [Magnus Heino](https://discuss.axoniq.io/u/daysleeper75), which started a discussion on our [forum](https://discuss.axoniq.io/t/events-other-than-first-event-created-by-contextawareeventmultiupcaster-are-not-processed-by-eventhandler/3756) after he noticed the issue.

### Release 4.5.6

* Although Axon Framework doesn't use the log4j-core dependency directly, we updated it to the most recent version for ease of mind.
  You can follow these increments in issues [#2038](https://github.com/AxonFramework/AxonFramework/pull/2038), [#2040](https://github.com/AxonFramework/AxonFramework/pull/2040) and [#2052](https://github.com/AxonFramework/AxonFramework/pull/2052).

* Contributor `jasperfect` spotted a predicament with duplicate aggregate creation combined with using caches.
  Axon didn't invalidate the cache as it should have, causing unexpected behavior.
  You can find the issue description [here](https://github.com/AxonFramework/AxonFramework/issues/2017).
  Additionally, you can find the pull request solving the problem [here](https://github.com/AxonFramework/AxonFramework/pull/2027).

* Contributor `shubhojitr` stated in issue [#2051](https://github.com/AxonFramework/AxonFramework/issues/2051) that the `axonserver-connector-java` project pulled in a non-secure version of `grpc-netty`.
  As this isn't an issue on Axon Framework itself, we solved the problem under the connector project.
  As a follow-up, we incremented the framework's version for the `axonserver-connector-java` project to 4.5.4, which contains the most recent version of the `grpc-bom`.

For an exhaustive list of all the changes, check out the [4.5.6 release notes](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.6).

### Release 4.5.5

* The auto-configuration we introduced for `XStream` used a suboptimal approach. 
  We assumed searching for the `@ComponentScan` would suffice but didn't consider that Spring enabled SpEL operations in the annotation's properties. 
  This approach thus caused some applications to break on start-up. 
  As such, this approach is replaced entirely by using the outcome of the `AutoConfigurationPackages#get(BeanFactory)` method. 
  For those interested in the details of the solution, check out [this](https://github.com/AxonFramework/AxonFramework/pull/1976) pull request. Kudos to contributor `maverick1601` for drafting issue [#1963](https://github.com/AxonFramework/AxonFramework/issues/1963) explaining the predicament.

* We introduced an optimization towards updating the `TrackingToken`. 
  In (distributed) environments where the configuration states several segments per Streaming Processor, there are always threads receiving events that they're not in charge of due to the configured `SequencingPolicy`. 
  The old implementation eagerly updated the token in such scenarios, but this didn't benefit the end-user immediately. 
  Pull request [#1999](https://github.com/AxonFramework/AxonFramework/pull/1999) introduce a wait period for 'event-less-batches', for both the `TrackingEventProcessor` and `PooledStreamingEventProcessor`. 
  This adjustment minimizes the number of token updates performed by both processor implementations.

* The introduction of Spring Boot version 2.6.0 brought an issue to light within Axon's Spring usage. 
  The `AbstractAnnotationHandlerBeanPostProcessor` took `FactoryBean` instances into account when searching for message handling methods. 
  This approach, however, is not recommended by Spring, which they enforced in their latest release. 
  The result was circular dependency exceptions on start-up whenever somebody used Spring Boot 2.6.0.
  The fix was simple, though, as we should simply ignore `FactoryBean` instances. 
  After spotting the issue, we resolved it in [this](https://github.com/AxonFramework/AxonFramework/pull/2013) pull request.

For an exhaustive list of all the changes, check out the [4.5.5 release notes](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.5).

### Release 4.5.4

* First and foremost, we updated the XStream version to 1.4.18. This upgrade was a requirement since several [CVE's](https://x-stream.github.io/changes.html) were noted for XStream version 1.4.17. 
  As a consequence of XStream's solution imposed through the CVE's, everybody is required to specify the security context of an `XStream` instance. 
  This change also has an impact on Axon Framework since the `XStreamSerializer` is the default serializer. 
  So as of this release, any usages of the default `XStreamSerializer` will come with warnings, stating it is highly recommended to use an `XStream` instance for which the security context is set through types or wildcards. 
  When your application uses Spring Boot, Axon will default to selecting the secured types based on your `@ComponentScan` annotated beans (e.g., like the `@SpringBootApplication` annotation). 
  For those interested in the details of the solution, check out [this](https://github.com/AxonFramework/AxonFramework/pull/1917) pull request.

* User 'nils-christian' noted in issue [#1892](https://github.com/AxonFramework/AxonFramework/issues/1892) that Axon executed Upcaster beans in a Spring environment in the incorrect order. 
  This ordering issue was due to a misconception in deducing the `@Order` annotation on upcaster beans. 
  We resolved the problem in pull request [#1895](https://github.com/AxonFramework/AxonFramework/pull/1895).

* We noticed a `TokenStore` operation that Axon did not invoke within a transaction. 
  In most scenarios, this worked out, but when using Micronaut, for example, this (correctly) caused an exception. 
  After spotting the issue, we resolved it in [this](https://github.com/AxonFramework/AxonFramework/pull/1908) pull request.

For an exhaustive list of all the changes, check out the [4.5.4 release notes](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.4).

### Release 4.5.3

* One new feature has been introduced in 4.5.3: the `PropertySequencingPolicy` by contributor `nils-christian`.
  This [sequencing policy](../../axon-framework/events/event-processors.md#sequential-processing) can be configured to look for a common property in the events.

* The version of the `axonserver-connector-java` has been updated to 4.5.2.
  This update resolves a troublesome issue around permit updates for subscription queries, which exhausted the number of queries an application could have running.
  For those curious about the solution, pull request [85](https://github.com/AxonIQ/axonserver-connector-java/pull/85) addresses this issue.

* The `WorkerLauncher` runnable, used by the `TrackingEventProcessor` to start its threads, was not considered when you shut down a tracking processor.
  As a consequence, it could start new segment operations while `shutdown` already completed "successfully."
  Pull request [1866](https://github.com/AxonFramework/AxonFramework/pull/1866) resolves this problem, ensuring a tracking processor shuts down as intended.

* Issue [1853](https://github.com/AxonFramework/AxonFramework/issues/1853) describes an issue where the [creation policy](../../axon-framework/axon-framework-commands/command-handlers.md#aggregate-command-handler-creation-policy) `always`.
  Exceptions thrown from within a command handler annotated with `@CreationPolicy(ALWAYS)` weren't correctly propagated.
  Pull request [1854](https://github.com/AxonFramework/AxonFramework/pull/1854) solves this issue.

For an exhaustive list of all the changes, check out the [4.5.3 release notes](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.3).

### Release 4.5.2

* Added a missing `isReplaying` flag on the `StreamingEventProcessor`.
Pull request [#1821](https://github.com/AxonFramework/AxonFramework/pull/1821) reintroduces this functionality in this release.

* Some enhancements in regards to logging Exceptions and stacktraces when initialization fails.
This [commit](https://github.com/AxonFramework/AxonFramework/commit/197eabea4259f98a4a06c999e4bd5ed7b373a3d4) reintroduces this functionality in this release.

* Improved Axon Framework (`AxonServerEventStore`) which will now rethrown Exceptions that has a valid `Status.Code`.
Pull request [#1842](https://github.com/AxonFramework/AxonFramework/pull/1842) reintroduces this functionality in this release.

* General improvements on the `PooledStreamingEventProcessor` made across several Pull Requests.

For a detailed perspective on the release notes, please check [this](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.2) page.

### Release 4.5.1

* Some internals have changed concerning command handling exceptions.
  Within a single JVM, Axon Framework knows whether the exception is transient or not.
  This piece of information allows the [`RetryScheduler`](../../axon-framework/axon-framework-commands/infrastructure.md#configuring-the-command-gateway) to retry a non-transient exception since those are retryable.
  With the move towards distributed environments, the information whether an exception is transient was lost when we moved to the dedicated `CommandHandlingException` containing a details object.
  Pull request [#1742](https://github.com/AxonFramework/AxonFramework/pull/1743) reintroduces this functionality in this release.

* The new `RevisionSnapshotFilter` introduced in release 4.5 sneaked in a bug by not validating the aggregate type upon filtering.
  Pull request [#1771](https://github.com/AxonFramework/AxonFramework/pull/1771) describes and solves the problem by introducing the aggregate type to the `RevisionSnapshotFilter`.

* By enabling the [`CreationPolicy`](../../axon-framework/axon-framework-commands/command-handlers.md#aggregate-command-handler-creation-policy) for the [`DisruptorCommandBus`](../../axon-framework/axon-framework-commands/infrastructure.md#disruptorcommandbus), a timing issue was introduced with handling events.
  Contributor "junkdog" marked the problem in issue [#1778](https://github.com/AxonFramework/AxonFramework/issues/1778), after which pull request [#1792](https://github.com/AxonFramework/AxonFramework/pull/1792) solved it.

* Contributor "michaelbub" noted in issue [#1786](https://github.com/AxonFramework/AxonFramework/issues/1786) that resetting a `StreamingEventProcessor` to a point in the future reacted differently when no token was stored yet.
  This followed from the implementation of the `ReplayToken`, which wrongfully assumed that if the given 'token at reset' was `null`, the start position should be `null` too.
  However, the start position might be the future, and hence it should be used in favor of `null`.
  This issue is addressed under [this](https://github.com/AxonFramework/AxonFramework/pull/1802) pull request.

For a detailed perspective on the release notes, please check [this](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.5.1) page.

## Release 4.4

### Release 4.4.9

Release 4.4.9 of Axon Framework has incremented _all_ used dependencies towards their latest bug release.
This has done to resolve potentially security issues, as was reported with XStream 1.4.14 (that was resolved in 1.4.16).

For those looking for the set of adjustments please take a look at tag [4.4.9](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.4.8)

### Release 4.4.8

* A bug was noted whenever a query handler returned a `Future`/`CompletableFuture` in combination with a subscription query, with Axon Server as the infrastructure.
  In this format, Axon would incorrectly use the scatter-gather query for the initial result of the subscription query.
  Whenever the returned result was completed, this didn't cause any issues. 
  However, for a `Future`/`CompletableFuture` a `TimeoutException` would be thrown.
  The issue was luckily easily mitigated by changing the "number of expected results" from within the `QueryRequest` to default to 1 instead of zero.
  As an effect, the point-to-point would be invoked instead of scatter-gather.
  For reference, the issue can be found [here](https://github.com/AxonFramework/AxonFramework/issues/1737).
  
* Whenever an interface is used as the type of an `@AggregateMember` annotated field, Axon would throw a `NullPointerException`.
  This is far from friendly, and has been changed towards an `AxonConfigurationException` in pull request [#1742](https://github.com/AxonFramework/AxonFramework/pull/1742).

Note that the named issues comprise the complete changelist for [Axon Framework 4.4.8](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.4.8). 

### Release 4.4.7

* The [Axon Server Connector Java](https://github.com/AxonIQ/axonserver-connector-java) version 4.4.7 has been included in this release as well. 
  As such, it's fixes (found [here](https://github.com/AxonIQ/axonserver-connector-java/releases/tag/4.4.7)) are thus also part of this release.

* Contributor "krosenvold" noticed that the SQL to retrieve a stream of events was performed twice in quick concession.
  The provided solution (in pull request [#1689](https://github.com/AxonFramework/AxonFramework/pull/1689)) would resolve this, but the problem was spotted to originate elsewhere.
  Commit [16b7152](https://github.com/AxonFramework/AxonFramework/commit/16b71529472ddb7345bd247ee5dd930dc6bd2206) saw an end to this occurrence by making a minor tweak in the `EmbeddedEventStore`.

* As rightfully noticed by user "pepperbob", there was a type discrepancy when reading events through a tracking token.
  An event would always become a `DomainEventMessage` when read through the `EventStorageEngine`, whereas it might originally have been a regular `EventMessage`.
  The problem has been fixed in commit [c61a95b](https://github.com/AxonFramework/AxonFramework/commit/c61a95bff14cda0ed3fea154747067560a670b4d).
  Furthermore, the entire description of the issue can be found [here](https://github.com/AxonFramework/AxonFramework/issues/1697).

* Through the use of the `AxonServerQueryBus`, a cancelled subscription query was wrongfully completed normally where it should complete exceptionally.
  This problem is marked and resolved under pull request [#1695](https://github.com/AxonFramework/AxonFramework/pull/1695).

For a detailed perspective on the release notes, please check [this](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.4.7) page. 

### Release 4.4.6

* Contributor "Rafaesp" noted that a registered `CommandHandlerInterceptor` in the `AggregateTestFixture` could be invoked more often than desired.
  This only occurred if the fixture's `givenCommands(...)` method was invoked, but nonetheless this behaviour was incorrect.
  The issue is marked under [#1665](https://github.com/AxonFramework/AxonFramework/issues/1665) and resolved in pull request [#1666](https://github.com/AxonFramework/AxonFramework/pull/1666).
  
* In 4.4.4, a fix was introduced which ensured a `ChildEntity` (read, the Aggregate Members) was no longer duplicated in an aggregate hierarchy.
  This fix had the troublesome side effect that aggregate member command handlers weren't registered on every level of the aggregate hierarchy anymore.
  The resolution to this problem can be found in pull request [#1674](https://github.com/AxonFramework/AxonFramework/pull/1674).
  
* Using the subscription query in a distributed environment had a possible troublesome side effect.
  If a consumer of updates was closed for whatever reason, it could also close the producing side. 
  This is obviously undesired, as no single consumer should influence if the producer should still dispatch updates to other consumers.
  The problem was marked under issue [#1680](https://github.com/AxonFramework/AxonFramework/issues/1680) and resolved in [this](https://github.com/AxonFramework/AxonFramework/commit/9907ae9bc1374a58ad9c8eca3dad2004086e2261) commit.
  
* Right before we aimed to release 4.4.6, contributor "haraldk" provided a thorough issue description when using the `SequenceEventStorageEngine`.
  He noted that if snapshots were used for an aggregate, there was a window of opportunity that the 'active' `EventStorageEngine` in the sequencing engine did not return any events.
  This followed from the sequence number logic, which wrongfully defaulted to position "0", even though the starting sequence number is per definition higher if a snapshot has been found.
  The clarifying issue can be found [here](https://github.com/AxonFramework/AxonFramework/issues/1682), with its resolution present in pull request [#1683](https://github.com/AxonFramework/AxonFramework/pull/1683).

For a complete overview of all the changes you can check the release notes [here](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.4.6).

### Release 4.4.5

* When creating a `TrackingToken` at a certain position through `StreamableMessageSource#createTokenAt(Instant)`, a tail token was wrongfully returned if the provided timestamp exceeded the timestamp of the last event.
  Instead, the token closests to the provided timestamp should be returned, was equals the head token.
  This discrepancy between documentation and implementation was marked by `mbreevoort` and resolved in pull request [#1607](https://github.com/AxonFramework/AxonFramework/pull/1607).

* A certain path within the `AxonServerEventStore` allowed for event retrieval without correctly deserializing the `MetaData` of the events.
  If someone tried to access the `MetaData`, a `CannotConvertBetweenTypesException` was being thrown.
  This problem, among others, was remedied in pull request [#1612](https://github.com/AxonFramework/AxonFramework/pull/1612), by ensuring the correct `Serializer` taking gRPC message types into account is consistently used.   

For a complete set of the release notes, please check [here](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.4.5).

### Release 4.4.4

* There was a bug which made it so that an `@ResetHandler` annotated method without any parameters was included for validation if a component could handle a specific type of event.
  This exact validation is used to filter out events from the event stream to optimize the entire stream.
  The optimization was thus mitigated by the simple fact of introducing a default `@ResetHandler`.
  The problem was marked by `@kad-hesseg` (for which thanks) and resolved in pull request [#1597](https://github.com/AxonFramework/AxonFramework/pull/1597).
 
 * A new `SnapshotTriggerDefinition` called `AggregateLoadTimeSnapShotTriggerDefinition` has been introduced, which uses the load time of an aggregate to trigger a snapshot creation.
 
 * When using an aggregate class hierarchy, `@AggregateMember` annotated fields present on the root would be duplicated for every class in the hierarchy which included message handling functions.
   This problem was traced back to the `AnnotatedAggregateMetaModelFactory.AnnotatedAggregateModel` which looped over an inconsistent set of classes to find these members.
   The issue was marked by `@kad-malota` and resolved in pull request [#1595](https://github.com/AxonFramework/AxonFramework/pull/1595).

For a complete set of the release notes, please check [here](https://github.com/AxonFramework/AxonFramework/releases/tag/axon-4.4.4).

### Release 4.4.3

* An optimization in the snapshotting process was introduced in pull request [#1510](https://github.com/AxonFramework/AxonFramework/pull/1510).
  This PR ensures no unnecessary snapshots are staged in the `AbstractSnapshotter` by validating none have been scheduled yet.
  This fix will resolve potential high I.O. when snapshots are being recreated for aggregates which have a high number of events.

* The assignment rules used by the `EventProcessingConfigurer` weren't always taken into account as desired.
  This inconsistency compared to regular assignment through the `@ProcessingGroup` annotation has been resolved in [this](https://github.com/AxonFramework/AxonFramework/pull/1500) pull request.
  
* Heartbeat messages between Axon Server and an Axon Framework application were already configurable, but only from the server's side.
  Properties have been introduced to also enables this from the clients end, as specified further in [this](https://github.com/AxonFramework/AxonFramework/pull/1511) pull request.
  Enabling heartbeat messages will ensure the connection is preemptively closed if no response has been received in the configured time frame.

To check out all fixes introduced in 4.4.3, you can check them out on [this](https://github.com/AxonFramework/AxonFramework/issues?q=is%3Aclosed+milestone%3A%22Release+4.4.3%22) page.

### Release 4.4.2

* A persistent loop of 500ms was spotted during event consumption from Axon Server.
  Credits go to Damir Murat who has spotted the [issue](https://github.com/AxonFramework/AxonFramework/issues/1481).
  With his help the issue was found quickly and eventually resolved in pull request [#1484](https://github.com/AxonFramework/AxonFramework/pull/1484).

* A serialization issue was found when working with the `ConfigToken` and de-/serialize it through the `JacksonSerializer`.
  This problem was uncovered in issue [#1482](https://github.com/AxonFramework/AxonFramework/issues/1482) and resolved in pull request [#1485](https://github.com/AxonFramework/AxonFramework/pull/1485).

* The introduction of the [AxonServer Connector for Java](https://github.com/AxonIQ/axonserver-connector-java) to simplify the framework's integration with Axon Server introduced some configuration issues.
  For example, the `AxonServerConfiguration#isForceReadFromLeader` wasn't used when opening an event stream (resolved in PR [#1488](https://github.com/AxonFramework/AxonFramework/pull/1488)).
  
* Furthermore, properties like the `max-message-size`, gRPC keep alive settings and `processorNotificationRate` weren't used when forming a connection with Axon Server.
  This issue was covered by pull request [#1487](https://github.com/AxonFramework/AxonFramework/pull/1487).

[This](https://github.com/AxonFramework/AxonFramework/issues?q=is%3Aclosed+milestone%3A%22Release+4.4.2%22) page shares a complete list of all resolved issues for this release.

### Release 4.4.1

A single fix was performed as soon as possible to release 4.4, in conjunction with the new [Axon Server Connector](https://github.com/AxonIQ/axonserver-connector-java) used by this release.
There was an off by one scenario when an Event Processor started reading events from the beginning of time.
This meant that the first event in the event store was systematically skipped.
The bug was resolved in [this](https://github.com/AxonFramework/AxonFramework/commit/3a055407437589bc1388cecca0b6e2f0bc61ea26) commit.

## Release 4.3

### Release 4.3.5

* The `TrackingEventProcessor#mergeSegment(int)` method was invoked with the high segment number of the pair to merge,

  an error would occur in the process as it expected to receive the lower number on all scenarios.

  This was resolved in pull request [\#1450](https://github.com/AxonFramework/AxonFramework/pull/1450).

* A small connectivity adjustment which was performed in the `AxonServerConnectionManager` for bug release 4.3.4 has been reverted.

  Although it worked successfully for some scenarios, it did not correctly cover all possibilities.

  The commit can be found [here](https://github.com/AxonFramework/AxonFramework/commit/5b9348040f4f977db3b9a15c3ae55904710814b6) for reference.

  The full scenario will be covered through the adjusted connector which is underway for beta release in 4.4.

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?q=is%3Aclosed+milestone%3A%22Release+4.3.5%22++label%3A%22Type%3A+Bug%22+).

### Release 4.3.4

* Whilst adjusting the `JdbcEventStorageEngine` in [\#1187](https://github.com/AxonFramework/AxonFramework/issues/1187) to allow more flexibility to configure the used statements, we accidentally dropped support for adjusting how the store wrote timestamps.

  This issue was rectified by user `ovstetun` in pull request [\#1454](https://github.com/AxonFramework/AxonFramework/pull/1454).

* Snapshots were incorrectly created in the same phase as the publication of events.

  This has been moved to the after commit phase of the `UnitOfWork` in issue [\#1457](https://github.com/AxonFramework/AxonFramework/pull/1457).

* When using the `SequenceEventStorageEngine` to merge an active and historic event stream there was a discrepancy when the active stream didn't contain any events and the historic stream did.

  This has been resolved in pull request [\#1459](https://github.com/AxonFramework/AxonFramework/pull/1459).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?q=is%3Aclosed+milestone%3A%22Release+4.3.4%22++label%3A%22Type%3A+Bug%22+).

### Release 4.3.3

This bug release contained a single fix, under pull request [\#1425](https://github.com/AxonFramework/AxonFramework/pull/1425). A situation was reported where a Tracking Event Processor did not catch up with the last event, until a new event was available after that event. Effectively causing it to read up to N-1. This only accounted for usages of the `MultiStreamableMessageSource`, thus when two \(or more\) event streams were combined into a single source for a `TrackingEventProcessor`.

To remain complete, [here](https://github.com/AxonFramework/AxonFramework/issues?q=is%3Aclosed+milestone%3A%22Release+4.3.3%22++label%3A%22Type%3A+Bug%22+) is the issue tracker page contained the closed issues for release 4.3.3.

### Release 4.3.2

* When using the `QueryGateway`, it was not possible to provide a `QueryMessage` as the query field since the `queryName` would be derived from the class name of the provided query.

  Hence, `QueryMessage` would be the `queryName`, instead of the actual `queryName`.

  This issue has been resolved in [\#1410](https://github.com/AxonFramework/AxonFramework/pull/1410).

* There was a window of opportunity where the `Snapshotter` would publish the last event in its stream twice.

  This could cause faulty snapshots in some scenarios.

  This issue was marked under [\#1408](https://github.com/AxonFramework/AxonFramework/issues/1408) and resolved in pull request [\#1416](https://github.com/AxonFramework/AxonFramework/pull/1416).

* The bi-directional stream created by the Axon Server Connector wasn't always closed correctly; specifically in error cases.

  This problem has been resolved in pull request [1397](https://github.com/AxonFramework/AxonFramework/pull/1397).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?q=is%3Aclosed+milestone%3A%22Release+4.3.2%22++label%3A%22Type%3A+Bug%22+).

### Release 4.3.1

* Through the new [Create-or-Update](../../axon-framework/axon-framework-commands/command-handlers.md#aggregate-command-handler-creation-policy)

  feature a bug was introduced which didn't allow non-String aggregate identifiers.

  This problem was quickly resolved in [\#1363](https://github.com/AxonFramework/AxonFramework/pull/1363),

  allowing the usage of "complex" aggregate identifiers once more.

* The graceful shutdown process introduced in 4.3 had a couple of minor problems.

  One of which was the shutdown order within the `AxonServerCommandBus` and `AxonServerQueryBus`,

  which basically made it so that the approach prior to 4.3 was maintained.

  We also noticed that the `AxonServerConnectionManager` never shutdown nicely.

  All of these, plus some other minor fixes, have been performed in [\#1372](https://github.com/AxonFramework/AxonFramework/pull/1372).

* The `AggregateCreationPolicy#ALWAYS` did not behave as expected, resulting in faulty behaviour when used.

  Pull request [\#1371](https://github.com/AxonFramework/AxonFramework/pull/1371) saw an end to this problem,

  ensuring the desired usage of all newly introduced creation policies.

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?q=is%3Aclosed+milestone%3A%22Release+4.3.1%22++label%3A%22Type%3A+Bug%22+).

## Release 4.2

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

## Release 4.1

### Release 4.1.2

* A dependency on `XStream` was enforced undesirably through the Builder pattern introduced in 4.0.

  This has been resolved by using a `Supplier` of a `Serializer` in the Builders instead, as described under [this](https://github.com/AxonFramework/AxonFramework/issues/1054) issue.

* Due to a hierarchy issue in the Spring Boot auto configuration, the `JdbcTokenStore` was not always used as expected.

  The ordering has been fixed under issue [\#1077](https://github.com/AxonFramework/AxonFramework/issues/1077).

* The ordering of message handling functions was incorrect according to the documentation.

  Classes take precedence over interface, and the depth of interface hierarchy is calculated based on the inheritance level \(as described [here](https://github.com/AxonFramework/AxonFramework/pull/1129)\).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.1.1

* Query Dispatch Interceptors were not called correctly when a [subscription query](../../axon-framework/queries/query-dispatchers.md#subscription-queries) was performed when Axon Server was used as the `QueryBus`.

  This issue was marked [here](https://github.com/AxonFramework/AxonFramework/issues/1013) and resolved in pull request [\#1042](https://github.com/AxonFramework/AxonFramework/pull/1042).

* When Axon Server was \(auto\) configured without being able to connect to an actual instance, processing instructions were incorrectly dispatched regardless.

  Pull request [\#1040](https://github.com/AxonFramework/AxonFramework/pull/1040) resolves this by making sure an active connection is present.

* The Spring Boot auto configuration did not allow the exclusion of the `axon-server-connector` dependency due to a direct dependency on classes.

  This has been resolved by expecting fully qualified class names as Strings instead \(resolved under [this](https://github.com/AxonFramework/AxonFramework/pull/1041) pull request\).

* The `JpaEventStorageEngine` was not wrapping the `appendEvents` operation in a transaction.

  Problem has been resolved under issue [\#1035](https://github.com/AxonFramework/AxonFramework/issues/1035).

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1.1%22++label%3A%22Type%3A+Bug%22).

## Release 4.0

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
* Command or Query handling exceptions were not properly serialized through Axon Server \(resolved in [\#904](https://github.com/AxonFramework/AxonFramework/pull/904)\). 

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.1

* Resolved `QueryUpdateEmitter` configuration for the Axon Server connector set up \(see issue [here](https://github.com/AxonFramework/AxonFramework/issues/896)\).
* For migration purposes legacy `TrackingTokens` should have been added, which is resolved [here](https://github.com/AxonFramework/AxonFramework/issues/886).
* Event Processing was stopped after a reconnection with Axon Server. Resolve the problem in issue [\#883](https://github.com/AxonFramework/AxonFramework/issues/883). 

For a complete list of all resolved bugs we refer to the [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.1%22++label%3A%22Type%3A+Bug%22).

