# Release Notes

All the enhancements and features which have been introduced to our major and minor release are documented here. This covers improvements to [Axon Framework](https://github.com/AxonFramework/AxonFramework), [Axon Server](https://axoniq.io/product-overview/axon-server) and the [Axon Framework Extensions](https://github.com/AxonFramework?utf8=%E2%9C%93&q=extensions&type=&language=), since all three follow the same release cadence. For bug fixes per project, we refer to [this](bug-fixes.md) page.

## Release 4.3

* Aggregate Polymorphism has been introduced, allowing for an aggregate hierarchy as would come natural from a domain model.

  The set this up, the `AggregateConfigurer#withSubtypes(Class... aggregates)` method can be used.

  In a Spring environment, an aggregate class hierarchy will be detected automatically.

  For more details on this feature, read up on it [here]().

* An Axon application will no shutdown more gracefully then it used to in previous releases.

  This is achieved by marking specific methods in Axon's infrastructure components as a `@StartHandler` or `@ShutdownHandler`.

  A 'phase' is required in those, specifying when the method should be executed.

  If you want to add your own lifecycle handlers,

   you can either register a component with the aforementioned annotations 

   or register the methods directly through `Configurer#onInitialize`, `Configuration#onStart` and

   `Configuration#onShutdown`.

* We have introduced the `@CreationPolicy` annotation which you can add to `@CommandHandler` annotated methods in your aggregate.

  Through this, it is possible te define if such a command handler should 'never', 'always' or 'create' an aggregate 'if-missing'.

  For further explanation read the [Aggregate Command Handler Creation Policy](../axon-framework/axon-framework-commands/command-handlers.md#aggregate-command-handler-creation-policy) section.

* Both the `XStreamSerializer` and `JacksonSerializer` provide a means to toggle on "lenient serialization" through their builders.
* Various test fixture improvements have been made, such as options to register a `HandlerEnhancerDefinition`,

  a `ParameterResolverFactory` and a `ListenerInvocationErrorHandler`.

  Additionally validations have been added too, revolving around asserting scheduled events and deadline message.

  The [Test Fixture](../axon-framework/testing/) page has been updated to define these new operations accordingly. 

* The `TrackingEventProcessor#processingStatus` method as of 4.3 exposes more status information.

  The current token position, token-at-reset, is-merging and merge-completed position have been added to the set.

  Read the [Event Tracker Status](../axon-framework/monitoring-and-metrics.md#event-tracker-status) section for more specifics on this. 

For a complete list of all the changes made in 4.3 you can check out [this](https://github.com/AxonFramework/AxonFramework/milestone/42?closed=1) page.

## Release 4.2

* Axon Framework applications can now use tags to support a level of 'location awareness' between Axon clients and Axon Server instances.

  This feature is further described [here](../axon-server/administration/tagging.md).

* Axon Server already supported several contexts, but Axon Framework application could no specify to which context message should be dispatched.

  The Axon Server Connector has been expanded with a `TargetContextResolver` to allow just this.

* A new implementation of the `StreamablbeMessageSource` has been implemented: the `MultiStreamableMessageSource`.

  This implementation allow pairing several "streamable" message sources into a single source.

  This can in turn be used to for example read events from several distinct contexts for a single Tracking Event Processor.

* Handler Execution Exception now allow application specific information to be sent back over the wire in case of a distributed set up.
* The `TrackingToken` interface now provides an estimate of it's relative position in the event stream through the `position()` method.
* `Optional` return types can now be used for Query Handling methods.  

For a full list of all features, enhancements and bugs, check out the [issue tracker](https://github.com/AxonFramework/AxonFramework/milestone/38?closed=1).

## Release 4.1

* The `TrackingEventProcessor` now has an API to split and merge `TrackingTokens` during runtime of an application.

  Axon Server has additions to the UI to split and merge a given Tracking Event Processor's tokens.

* Next to [Dropwizard](https://metrics.dropwizard.io/4.0.0/) metrics the framework now also supports [Micrometer](https://micrometer.io/) metrics.

  The `MessageMonitor` interface is used to allow integration with Micrometer.

  Lastly, we are incredibly thankful that this has been introduced as a community contribution.

* Primitive types are now supported as `@QueryHandler` return types.
* In a similar fashion as the `CommandGateway` and `QueryGateway` we have introduced the `EventGateway`.

  As with the command and query version, the `EventGateway` provides a simpler API when it comes to dispatching Events on the `EventBus`.

We refer to [this](https://github.com/AxonFramework/AxonFramework/milestone/31?closed=1) page for a full list of all the changes.

## Release 4.0

* The package structure of Axon Framework has changed drastically with the aim to provide users the option to pick and choose.

  For example, if only the messaging components of framework are required, one can directly depend on the `axon-messaging` package.

* In part with the package restructure, all components which leverage another framework to provide something extra have been given their own repository.

  These repositories are called the [Axon Framework Extensions](https://github.com/AxonFramework?utf8=%E2%9C%93&q=extensions&type=&language=).

* The configuration of Event Processor has been replaced and greatly fine tuned with the addition of the `EventProcessingConfigurer`.     
* Some new defaults have been introduced in release 4.0, like a bias towards expecting a connection with Axon Server.

  Another important chance is the switch from defaulting to Tracking Processors instead of Subscribing Processors.

* The notion of a `CommandResultMessage` has been introduced as a dedicated message towards the result of command handling.
* To simplify configuration and more easily overcome deprecation,

   the [Builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) has been implemented for all infrastructure components.

For more details, check the list of issues [here](https://github.com/AxonFramework/AxonFramework/milestone/28?closed=1).

