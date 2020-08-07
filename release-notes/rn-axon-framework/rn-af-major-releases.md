# Major Releases

All the enhancements and features which have been introduced to our major releases of the Axon Framework are noted here.

## Release 4.4

### _Enhancements_

* Axon Framework can now be used in conjunction with [Spring Boot Developer Tools](https://docs.spring.io/spring-boot/docs/1.5.16.RELEASE/reference/html/using-boot-devtools.html).
  You can simply achieve this by adding the required dev-tools dependency to your project.

* As a partial solution to [#1106](https://github.com/AxonFramework/AxonFramework/issues/1106), Axon Server can now be used to schedule events.
  Building an `AxonServerEventScheduler` as the `EventScheduler` implementation as defined through the builder is sufficient to start with scheduling events through Axon Server.
  
* An `EventTrackerStatusChangeListener` can now be configured for a `TrackingEventProcessor`, as was requested in [#1338](https://github.com/AxonFramework/AxonFramework/issues/1338).
  It can be configured through the `TrackingEventProcessorConfiguration`, allowing users to react upon status changes of each thread processing event messages.

* Component specific message handler interceptors can now be defined through a dedicated annotation: the `@MessageHandlerInterceptor` annotation.
  This annotation allows you to introduce a specific bit of logic to be invoked _prior_ to entering the message handling function or after invocation.
  It for example allows the additional introduction of a `@ExceptionHandler` annotation, allowing you to specifically deal with the exceptions thrown from your message handlers.
  The original pull request can be found under [#1394](https://github.com/AxonFramework/AxonFramework/pull/1394).
  For more specifics on using this annotation, check ou the [@MessageHandlerInterceptor](../../axon-framework/messaging-concepts/message-intercepting.md#messagehandlerinterceptor) section.

* Configuring a `Snapshotter` and `SnapshotFilter` have been simplified in this release. 
  Pull request [#1447](https://github.com/AxonFramework/AxonFramework/pull/1447) shares the load of allowing for distinct `Snapshotter` configuration.
  Issue [#1391](https://github.com/AxonFramework/AxonFramework/issues/1391) describes the intent to the configuration of snapshot filtering to be performed on Aggregate level.
  The former can be configured through the `Configurer`, whereas the latter is by usage of the `AggregateConfigurer`.

### _Bug Fixes_

* The `AggregateTestFixture` was incorrectly noting an old method in one of its exceptions.
  This has been marked and resolved in [#1428](https://github.com/AxonFramework/AxonFramework/issues/1428).

* The `CommandValidator` and `EventValidator` had a minor discrepancy; namely, the `CommandValidator` cleared out contained commands upon starting whereas the `EventValidator` didn't.
  Pull request [#1438](https://github.com/AxonFramework/AxonFramework/pull/1438) resolved the problem at hand. 

For a full list of all the feature request and enhancements done for release 4.4, you can check out [this](https://github.com/AxonFramework/AxonFramework/milestone/45?closed=1) page.

## Release 4.3

### _Enhancements_

* Aggregate Polymorphism has been introduced, allowing for an aggregate hierarchy to come naturally from a domain model.
  To set this up, the `AggregateConfigurer#withSubtypes(Class... aggregates)` method can be used.
  In a Spring environment, an aggregate class hierarchy will be detected automatically.
  For more details on this feature, read up on it [here](../../axon-framework/axon-framework-commands/modeling/aggregate-polymorphism.md).

* An Axon application will now shutdown more gracefully than it did in the previous releases.
  This is achieved by marking specific methods in Axon's infrastructure components as a `@StartHandler` or `@ShutdownHandler`.
  A 'phase' is required in those, specifying when the method should be executed.
  If you want to add your own lifecycle handlers, you can either register a component with the aforementioned annotations or register the methods directly through `Configurer#onInitialize`, `Configuration#onStart` and `Configuration#onShutdown`.

* We have introduced the `@CreationPolicy` annotation which you can add to `@CommandHandler` annotated methods in your aggregate.
  Through this, it is possible to define if such a command handler should 'never', 'always' or 'create' an aggregate 'if-missing'.
  For further explanation read the [Aggregate Command Handler Creation Policy](../../axon-framework/axon-framework-commands/command-handlers.md#aggregate-command-handler-creation-policy) section.

* Both the `XStreamSerializer` and `JacksonSerializer` provide a means to toggle on "lenient serialization" through their builders.

* Various test fixture improvements have been made, such as options to register a `HandlerEnhancerDefinition`, a `ParameterResolverFactory` and a `ListenerInvocationErrorHandler`.
  Additionally validations have been added too, revolving around asserting scheduled events and deadline message.
  The [Test Fixture](../../axon-framework/testing/commands-events.md) page has been updated to define these new operations accordingly.

* The `TrackingEventProcessor#processingStatus` method as of 4.3 exposes more status information.
  The current token position, token-at-reset, is-merging and merge-completed position have been added to the set.
  Read the [Event Tracker Status](../../axon-framework/monitoring-and-metrics.md#event-tracker-status-a-idevent-tracker-statusa) section for more specifics on this.

### _Bug Fixes_

* A `ConcurrencyException` was thrown when an aggregate was created at two distinct JVM's at the same time.
  As `ConcurrencyException`s are typically retryable,  the creation command would be issued again if a `RetryScheduler` was in place.
  Retrying this operation is however useless and hence has been replaced for an `AggregateStreamCreationException` in pull request [\#1333](https://github.com/AxonFramework/AxonFramework/pull/1333).

* The test fixtures for state-stored aggregates did unintentionally not allow resource injection.
  This problem has been resolved in pull request [\#1315](https://github.com/AxonFramework/AxonFramework/pull/1315).

* The `MultiStreamableMessageSource` did not deal well with one or several exceptional streams.
  Hence exception handling has been improved on this matter in [\#1325](https://github.com/AxonFramework/AxonFramework/pull/1325).

For a complete list of all the changes made in 4.3 you can check out [this](https://github.com/AxonFramework/AxonFramework/milestone/42?closed=1) page.

## Release 4.2

### _Enhancements_

* Axon Framework applications can now use tags to support a level of 'location awareness' between Axon clients and Axon Server instances.
  This feature is further described [here](../../axon-server/administration/tagging.md).

* Axon Server already supported several contexts, but Axon Framework application could not specify to which context message should be dispatched.
  The Axon Server Connector has been expanded with a `TargetContextResolver` to allow just this.

* A new implementation of the `StreamablbeMessageSource` has been implemented: the `MultiStreamableMessageSource`.
  This implementation allows pairing several "streamable" message sources into a single source.
  This can in turn be used to for example read events from several distinct contexts for a single Tracking Event Processor.

* Handler Execution Exception now allow application specific information to be sent back over the wire in case of a distributed set up.

* The `TrackingToken` interface now provides an estimate of it's relative position in the event stream through the `position()` method.

* `Optional` return types can now be used for Query Handling methods.  

### _Bug Fixes_

* An Aggregate's `Snapshotter` was not auto configured when Spring Boot is being used, as was filed under [\#932](https://github.com/AxonFramework/AxonFramework/issues/932).

* The `CommandResultMessage` was returned as `null` when using the [`DisruptorCommandBus`](./).
  This was solved in pull request [\#1169](https://github.com/AxonFramework/AxonFramework/pull/1169).

* The `ScopeDescriptor` used by the `DeadlineManager` had serialization issue when a user would migrate from an Axon 3.x application to Axon 4.x.
  The `axon-legacy` package has been expanded to contain legacy `ScopeDescriptor`s to resolve this problem.

For a full list of all features, enhancements and bugs, check out the [issue tracker](https://github.com/AxonFramework/AxonFramework/milestone/38?closed=1).

## Release 4.1

### _Enhancements_

* The `TrackingEventProcessor` now has an API to split and merge `TrackingTokens` during runtime of an application.
  Axon Server has additions to the UI to split and merge a given Tracking Event Processor's tokens.

* Next to [Dropwizard](https://metrics.dropwizard.io/4.0.0/) metrics the framework now also supports [Micrometer](https://micrometer.io/) metrics.
  The `MessageMonitor` interface is used to allow integration with Micrometer.
  Lastly, we are incredibly thankful that this has been introduced as a community contribution.

* Primitive types are now supported as `@QueryHandler` return types.

* We have introduced the `EventGateway` in a similar fashion as the `CommandGateway` and `QueryGateway`.
  As with the command and query version, the `EventGateway` provides a simpler API when it comes to dispatching Events on the `EventBus`.

### _Bug Fixes_

* Command and Query message priority was not set correctly for the Axon Server Connector.
  This issue has been addressed under bug [\#1004](https://github.com/AxonFramework/AxonFramework/pull/1004).

* The `CapacityMonitor` was not registered correctly for Event Processor, which user "Sabartius" resolved under issue [\#961](https://github.com/AxonFramework/AxonFramework/issues/961).

* Some exception were not reported correctly and/or clearly when utilizing the Axon Server Connector \(issue marked under number [\#945](https://github.com/AxonFramework/AxonFramework/pull/945)\).

We refer to [this](https://github.com/AxonFramework/AxonFramework/milestone/31?closed=1) page for a full list of all the changes.

## Release 4.0

### _Enhancements_

* The package structure of Axon Framework has changed drastically with the aim to provide users the option to pick and choose.
  For example, if only the messaging components of framework are required, one can directly depend on the `axon-messaging` package.

* In part with the package restructure, all components which leverage another framework to provide something extra have been given their own repository.
  These repositories are called the [Axon Framework Extensions](https://github.com/AxonFramework?utf8=%E2%9C%93&q=extensions&type=&language=).

* The configuration of Event Processor has been replaced and greatly fine tuned with the addition of the `EventProcessingConfigurer`.

* Some new defaults have been introduced in release 4.0, like a bias towards expecting a connection with Axon Server.
  Another important chance is the switch from defaulting to Tracking Processors instead of Subscribing Processors.

* The notion of a `CommandResultMessage` has been introduced as a dedicated message towards the result of command handling.

* To simplify configuration and more easily overcome deprecation, the [Builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) has been implemented for all infrastructure components.

### _Bug Fixes_

The bugs marked for release 4.0 were issues introduced to new features or enhancements. As such they should not have impacted users in any way. Regardless, the full list can be found [here](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0%22++label%3A%22Type%3A+Bug%22).

For more details, check the list of issues [here](https://github.com/AxonFramework/AxonFramework/milestone/28?closed=1).
