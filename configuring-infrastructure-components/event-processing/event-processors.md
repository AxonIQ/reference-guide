# Event Processors

Event Handlers define the business logic to be performed when an Event is received. Event Processors are the components that take care of the technical aspects of that processing. They start a Unit of Work and possibly a transaction, but also ensure that correlation data can be correctly attached to all messages created during Event processing.

Event Processors come in roughly two forms: Subscribing and Tracking. The Subscribing Event Processors subscribe themselves to a source of Events and are invoked by the thread managed by the publishing mechanism. Tracking Event Processors, on the other hand, pull their messages from a source using a thread that it manages itself.

## Assigning handlers to processors

All processors have a name, which identifies a processor instance across JVM instances. Two processors with the same name, can be considered as two instances of the same processor.

All Event Handlers are attached to a Processor whose name is the package name of the Event Handler's class.

For example, the following classes:

* `org.axonframework.example.eventhandling.MyHandler`,
* `org.axonframework.example.eventhandling.MyOtherHandler`, and
* `org.axonframework.example.eventhandling.module.MyHandler`

will trigger the creation of two Processors:

* `org.axonframework.example.eventhandling` with 2 handlers, and 
* `org.axonframework.example.eventhandling.module` with a single handler

The Configuration API allows you to configure other strategies for assigning classes to processors, or even assign specific instances to specific processors.

## Ordering Event Handlers within a single Event Processor

To order Event Handlers within an Event Processor, the ordering in which Event Handlers are registered \(as described in the [Registering Event Handlers](../1.2-domain-logic/event-handling.md#registering-event-handlers) section\) is guiding. Thus, the ordering in which Event Handlers will be called by an Event Processor for Event Handling is their insertion ordering in the configuration API.

If Spring is selected as the mechanism to wire everything, the ordering of the Event Handlers can be specified by adding the `@Order` annotation. This annotation should be placed on class level of your Event Handler class, adding a `integer` value to specify the ordering.

Do note that it is not possible to order Event Handlers which are not a part of the same Event Processor.

## Configuring processors

Processors take care of the technical aspects of handling an event, regardless of the business logic triggered by each event. However, the way "regular" \(singleton, stateless\) event handlers are Configured is slightly different from Sagas, as different aspects are important for both types of handlers.

### Event Handlers

By default, Axon will use Tracking Event Processors. It is possible to change how Handlers are assigned and how processors are configured using the `EventProcessingConfigurer` class of the Configuration API.

The `EventProcessingConfigurer` class defines a number of methods that can be used to define how processors need to be configured.

* `registerEventProcessorFactory` allows you to define a default factory method that creates Event Processors for which no explicit factories have been defined.
* `registerEventProcessor(String name, EventProcessorBuilder builder)` defines the factory method to use to create a Processor with given `name`. Note that such Processor is only created if `name` is chosen as the processor for any of the available Event Handler beans.
* `registerTrackingEventProcessor(String name)` defines that a processor with given name should be configured as a Tracking Event Processor, using default settings. It is configured with a TransactionManager and a TokenStore, both taken from the main configuration by default.
* `registerTrackingProcessor(String name, Function<Configuration, StreamableMessageSource<TrackedEventMessage<?>>> source, Function<Configuration, TrackingEventProcessorConfiguration> processorConfiguration)` defines that a processor with given name should be configured as a Tracking Processor, and use the given `TrackingEventProcessorConfiguration` to read the configuration settings for multi-threading. The `StreamableMessageSource` defines an event source from which this processor should pull for events.
* `usingSubscribingEventProcessors()` sets the default to subscribing event processors instead of tracking ones.

### Sagas

Sagas are configured using the `SagaConfigurer` class. 
It provides convenient methods for configuring a saga repository, a saga manager and a saga store.

Let's configure custom saga repository, saga manager and saga store:

```java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.eventProcessing(eventProcessingConfigurer -> 
                                eventProcessingConfigurer.registerSaga(MySaga.class,
                                                                       sagaConfigurer -> sagaConfigurer.configureSagaStore(c -> sagaStore)
                                                                                                       .configureRepository(c -> repository)
                                                                                                       .configureSagaManager(c -> manager)));
```

Check out the API documentation \(JavaDoc\) of the `SagaConfiguration` class for full details on how to configure event handling for a saga.

## Token Store

Tracking event processors, unlike subscribing ones, need a token store to store their progress in. Each message a tracking processor receives through its event stream is accompanied by a token. This token allows the processor to reopen the stream at any later point, picking up where it left off with the last Event.

The Configuration API takes the token store, as well as most other components processors need from the global configuration instance. If no token store is explicitly defined, an `InMemoryTokenStore` is used, which is _not recommended in production_.

To configure a different token store, use `Configurer.registerComponent(TokenStore.class, conf -> ... create token store ...)`

Note that you can override the token store to use with tracking processors in the respective `EventProcessingConfiguration` or `SagaConfiguration` that defines that processor. Where possible, it is recommended to use a token store that stores tokens in the same database as where the event handlers update the view models. This way, changes to the view model can be stored atomically with the changed tokens, guaranteeing exactly once processing semantics.

## Event Tracker Status

In some cases it might be useful to know the state of a Tracking Event Processor for each of its segment. One of those cases could be when we want to rebuild our view model and we want to check when the Processor is caught up with all the events. For cases like these, the `TrackingEventProcessor` exposes `processingStatus()` method, which returns a map where the key is the segment identifier, and the value is the event processing status. Based on this status we can determine whether the Processor is caught up and/or is replaying, and we can verify the Tracking Token of its segments.

## Parallel processing

Tracking processors can use multiple threads to process an event stream. They do so, by claiming a so-called segment, identifier by a number. Normally, a single thread will process a single segment.

The number of segments used can be defined. When a processor starts for the first time, it can initialize a number of segments. This number defines the maximum number of threads that can process events simultaneously. Each node running of a tracking processor will attempt to start its configured amount of threads, to start processing these.

Event handlers may have specific expectations on the ordering of events. If this is the case, the processor must ensure these events are sent to these handlers in that specific order. Axon uses the `SequencingPolicy` for this. The `SequencingPolicy` is essentially a function, that returns a value for any given message. If the return value of the `SequencingPolicy` function is equal for two distinct event messages, it means that those messages must be processed sequentially. By default, Axon components will use the `SequentialPerAggregatePolicy`, which makes it so that events published by the same aggregate instance will be handled sequentially.

A saga instance is never invoked concurrently by multiple threads. Therefore, a sequencing policy for a saga is irrelevant. Axon will ensure each saga instance receives the events it needs to process in the order they have been published on the event bus.

> **Note**
>
> Note that subscribing processors don't manage their own threads. Therefore, it is not possible to configure how they should receive their events. Effectively, they will always work on a sequential-per-aggregate basis, as that is generally the level of concurrency in the Command Handling component.

### Multi-node processing

For tracking processors, it doesn't matter whether the threads handling the events are all running on the same node, or on different nodes hosting the same \(logical\) tracking processor. When two instances of t tracking processor, having the same name, are active on different machines, they are considered two instances of the same logical processor. They will 'compete' for segments of the event stream. Each instance will 'claim' a segment, preventing events assigned to that segment from being processed on the other nodes.

The `TokenStore` instance will use the JVM's name \(usually a combination of the host name and process ID\) as the default `nodeId`. This can be overridden in `TokenStore` implementations that support multi-node processing.

## Distributing Events

Distributing events in inter-process environments is supported with Axon Server by default.

Alternatively, you can choose other components that you can find in one of the extension modules (Spring AMQP, Kafka).

## Asynchronous Event Processing

The recommended approach to handle events asynchronously is by using a tracking event processor. This implementation can guarantee processing of all events, even in case of a system failure \(assuming the events have been persisted\).

However, it is also possible to handle events asynchronously in a `SubscribingEventProcessor`. To achieve this, the `SubscribingEventProcessor` must be configured with an `EventProcessingStrategy`. This strategy can be used to change how invocations of the event listeners should be managed.

The default strategy \(`DirectEventProcessingStrategy`\) invokes these handlers in the thread that delivers the events. This allows processors to use existing transactions.

The other Axon-provided strategy is the `AsynchronousEventProcessingStrategy`. It uses an `Executor` to asynchronously invoke the èvent listeners.

Even though the `AsynchronousEventProcessingStrategy` executes asynchronously, it is still desirable that certain events are processed sequentially. The `SequencingPolicy` defines whether events must be handled sequentially, in parallel or a combination of both. Policies return a sequence identifier of a given event. If the policy returns an equal identifier for two events, this means that they must be handled sequentially by the event handler. A `null` sequence identifier means the event may be processed in parallel with any other event.

Axon provides a number of common policies you can use:

* The `FullConcurrencyPolicy` will tell Axon that this event handler may handle all events concurrently. This means that there is no relationship between the events that require them to be processed in a particular order.
* The `SequentialPolicy` tells Axon that all events must be processed sequentially. Handling of an event will start when the handling of a previous event is finished.
* `SequentialPerAggregatePolicy` will force domain events that were raised from the same aggregate to be handled sequentially. However, events from different aggregates may be handled concurrently. This is typically a suitable policy to use for event listeners that update details from aggregates in database tables.

Besides these provided policies, you can define your own. All policies must implement the `SequencingPolicy` interface. This interface defines a single method, `getSequenceIdentifierFor`, that returns the sequence identifier for a given event. Events for which an equal sequence identifier is returned must be processed sequentially. Events that produce a different sequence identifier may be processed concurrently. For performance reasons, policy implementations should return `null` if the event may be processed in parallel to any other event. This is faster, because Axon does not have to check for any restrictions on event processing.

It is recommended to explicitly define an `ErrorHandler` when using the `AsynchronousEventProcessingStrategy`. The default `ErrorHandler` propagates exceptions, but in an asynchronous execution, there is nothing to propagate to, other than the executor. This may result in Events not being processed. Instead, it is recommended to use an `ErrorHandler` that reports errors and allows processing to continue. The `ErrorHandler` is configured on the constructor of the `SubscribingEventProcessor`, where the `EventProcessingStrategy` is also provided.

## Replaying events

In cases when you want to rebuild projections \(view models\), replaying past events comes in handy. The idea is to start from the beginning of time and invoke all event handlers anew. The `TrackingEventProcessor` supports replaying of events. In order to achieve that, you should invoke the `resetTokens()` method on it. It is important to know that the \`tracking event processor must not be in active state when starting a reset. Hence it is wise to shut it down first, then reset it and once this was successful, start it up again. It is possible to define a `@ResetHandler`, so you can do some preparation prior to resetting. Let's take a look how we can accomplish replaying. First, we will see one simple projecting class.

```java
@ProcessingGroup("projections")
public class MyProjection {
    ...
    @EventHandler
    public void on(MyEvent event, ReplayStatus replayStatus) { 
                // we can wire a ReplayStatus here so we can see whether this
                // event is delivered to our handler as a 'REGULAR' event or
                // 'REPLAY' event
        // do event handling
    }

    @AllowReplay(false) // it is possible to prevent some handlers 
                        // from being replayed
    @EventHandler
    public void on(MyOtherEvent event) {
        // perform some side effect introducing functionality, 
        //  like sending an e-mail, which we do not want to be replayed
    }    

    @ResetHandler
    public void onReset() { // will be called before replay starts
        // do pre-reset logic, like clearing out the Projection table for a 
        // clean slate
    }
    ...
}
```

And now, we can reset our `TrackingEventProcessor`:

```java
configuration.eventProcessingConfiguration()
             .eventProcessorByProcessingGroup("projections", 
                                              TrackingEventProcessor.class)
             .ifPresent(trackingEventProcessor -> {
                 trackingEventProcessor.shutDown();
                 trackingEventProcessor.resetTokens(); // (1)
                 trackingEventProcessor.start();
             });
```

> **Note**
>
> It is possible to provide a token position to be used when resetting a `TrackingEventProcessor`, thus specifying from which point in the event log it should start replaying the events.

## Custom tracking token position

Prior to Axon release 3.3, you could only reset a `TrackingEventProcessor` to the beginning of the event stream. As of version 3.3 functionality for starting a `TrackingEventProcessor` from a custom position has been introduced. The `TrackingEventProcessorConfiguration` provides the option to set an initial token for a given `TrackingEventProcessor` through the `andInitialTrackingToken(Function<StreamableMessageSource, TrackingToken>)` builder method. As an input parameter for the token builder function, we receive a `StreamableMessageSource` which gives us three possibilities to build a token:

* From the head of event stream: `createHeadToken()`. 
* From the tail of event stream: `createTailToken()`. 
* From some point in time: `createTokenAt(Instant)` and `createTokenSince(duration)` - Creates a token that tracks all events after given time. If there is an event exactly at the given time, it will be taken into account too.

Of course, you can completely disregard the `StreamableMessageSource` input parameter and create a token by yourself.

Below we can see an example of creating a `TrackingEventProcessorConfiguration` with an initial token on `"2007-12-03T10:15:30.00Z"`:

```java
TrackingEventProcessorConfiguration
            .forSingleThreadedProcessing()
            .andInitialTrackingToken(
                streamableMessageSource -> 
                    streamableMessageSource.createTokenAt("2007-12-03T10:15:30.00Z"));
```