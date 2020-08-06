# Event Processors

Event handlers define the business logic to be performed when an event is received. Event processors are the components that take care of the technical aspects of that processing. They start a unit of work and possibly a transaction. However, they also ensure that correlation data can be correctly attached to all messages created during event processing.

A representation of the organization of Event Processors and Event Handlers is depicted below.

![Organization of Event Processors and Event Handlers](../../.gitbook/assets/event-processors.png)

Event Processors come in roughly two forms: Subscribing and Tracking. Subscribing Event Processors subscribe themselves to a source of Events and are invoked by the thread managed by the publishing mechanism. Tracking Event Processors, on the other hand, pull their messages from a source using a thread that it manages itself.

## Assigning handlers to processors

All processors have a name, which identifies a processor instance across JVM instances. Two processors with the same name are considered as two instances of the same processor.

All event handlers are attached to a processor whose name is the package name of the Event Handler's class.

For example, the following classes:

* `org.axonframework.example.eventhandling.MyHandler`,
* `org.axonframework.example.eventhandling.MyOtherHandler`, and
* `org.axonframework.example.eventhandling.module.MyHandler`

will trigger the creation of two processors:

* `org.axonframework.example.eventhandling` with 2 handlers, and
* `org.axonframework.example.eventhandling.module` with a single handler

The Configuration API allows you to configure other strategies for assigning classes to processors, or even assign specific handler instances to specific processors.

## Ordering Event Handlers within a single Event Processor

To order Event Handlers within an Event Processor, the order in which Event Handlers are registered \(as described in the [Registering Event Handlers](event-handlers.md#registering-event-handlers) section\) is guiding. Thus, the ordering in which Event Handlers will be called by an Event Processor for Event Handling is the same as their insertion ordering in the configuration API.

If Spring is selected as the mechanism to wire everything, the ordering of the Event Handlers can be explicitly specified by adding the `@Order` annotation. This annotation should be placed at the class level of your event handler class, and an `integer` value should be provided to specify the ordering.

Not that it is not possible to order event handlers which are not a part of the same event processor.

## Configuring processors

Processors take care of the technical aspects of handling an event, regardless of the business logic triggered by each event. However, the way "regular" \(singleton, stateless\) event handlers are configured is slightly different from sagas, as different aspects are important for both types of handlers.

### Event Processors

By default, Axon will use Tracking Event Processors. It is possible to change how handlers are assigned and how processors are configured.

{% tabs %}
{% tab title="Axon Configuration API" %}
The `EventProcessingConfigurer` class defines a number of methods that can be used to define how processors need to be configured.

* `registerEventProcessorFactory` allows you to define a default factory method that creates event processors for which no explicit factories have been defined.
* `registerEventProcessor(String name, EventProcessorBuilder builder)` defines the factory method to use to create a processor with given `name`.

  Note that such a processor is only created if `name` is chosen as the processor for any of the available event handler beans.

* `registerTrackingEventProcessor(String name)` defines that a processor with given name should be configured as a tracking event processor, using default settings.

  It is configured with a TransactionManager and a TokenStore, both taken from the main configuration by default.

* `registerTrackingProcessor(String name, Function<Configuration, StreamableMessageSource<TrackedEventMessage<?>>> source, Function<Configuration, TrackingEventProcessorConfiguration> processorConfiguration)` defines that a processor with given name should be configured as a tracking processor, and use the given `TrackingEventProcessorConfiguration` to read the configuration settings for multi-threading.

  The `StreamableMessageSource` defines an event source from which this processor should pull events.

* `usingSubscribingEventProcessors()` sets the default to subscribing event processors instead of tracking ones.

```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
    .eventProcessing(eventProcessingConfigurer -> eventProcessingConfigurer.usingSubscribingEventProcessors())
    .eventProcessing(eventProcessingConfigurer -> eventProcessingConfigurer.registerEventHandler(c -> new MyEventHandler()));
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
 // Default all processors to subscribing mode.
@Autowired
public void configure(EventProcessingConfigurer config) {
    config.usingSubscribingEventProcessors();
}
```

Certain aspects of event processors can also be configured in `application.properties`.

```text
axon.eventhandling.processors.name.mode=subscribing
axon.eventhandling.processors.name.source=eventBus
```

If the name of an event processor contains periods `.`, use the map notation:

```text
axon.eventhandling.processors[name].mode=subscribing
axon.eventhandling.processors[name].source=eventBus
```
{% endtab %}
{% endtabs %}

### Multiple Event Sources

You can configure a Tracking Event Processor to use multiple sources when processing events. This is useful for compiling metrics across domains or simply when your events are distributed between multiple event stores.

Having multiple sources means that there might be a choice of multiple events that the processor could consume at any given instant. Therefore, you can specify a `Comparator` to choose between them. The default implementation chooses the event with the oldest timestamp \(i.e. the event waiting the longest\).

Multiple sources also means that the tracking processor's polling interval needs to be divided between sources using a strategy to optimize event discovery and minimize overhead in establishing costly connections to the data sources. Therefore, you can choose which source the majority of the polling is done on using the `longPollingSource()` method in the builder. This ensures one source consumes most of the polling interval whilst also checking intermittently for events on the other sources. The default `longPollingSource` is done on the last configured source.

{% tabs %}
{% tab title="Axon Configuration API" %}
Create a `MultiStreamableMessageSource` using its `builder()` and register it as the message source when calling `EventProcessingConfigurer.registerTrackingEventProcessor()`.

For example:

```java
MultiStreamableMessageSource.builder()
        .addMessageSource("eventSourceA", eventSourceA)
        .addMessageSource("eventSourceB", eventSourceB)
        .longPollingSource("eventSourceA") // Overrides eventSourceB as the longPollingStream
        .trackedEventComparator(priorityA) // Where 'priorityA' is a comparator prioritizing events from eventSourceA
        .build();
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Bean
public MultiStreamableMessageSource multiStreamableMessageSource(EventStore eventSourceA, EventStore eventStoreB) {
    return MultiStreamableMessageSource.builder()
            .addMessageSource("eventSourceA", eventSourceA)
            .addMessageSource("eventSourceB", eventSourceB)
            .longPollingSource("eventSourceA") // Overrides eventSourceB as the longPollingStream
            .trackedEventComparator(priorityA) // Where 'priorityA' is a comparator prioritizing events from eventSourceA
            .build();
}

@Autowired
public void configure(EventProcessingConfigurer config, MultiStreamableMessageSource multiStreamableMessageSource) {
    config.registerTrackingEventProcessor("NameOfEventProcessor", c -> multiStreamableMessageSource);
}
```
{% endtab %}
{% endtabs %}

### Sagas

It is possible to change how Sagas are configured.

{% tabs %}
{% tab title="Axon Configuration API" %}
Sagas are configured using the `SagaConfigurer` class:

```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
    .eventProcessing(eventProcessingConfigurer -> 
        eventProcessingConfigurer.registerSaga(
            MySaga.class,
            sagaConfigurer -> sagaConfigurer.configureSagaStore(c -> sagaStore)
                .configureRepository(c -> repository)
                .configureSagaManager(c -> manager)
        )
    );
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
The configuration of infrastructure components to operate sagas is triggered by the `@Saga` annotation \(in the `org.axonframework.spring.stereotype` package\). Axon will then configure a `SagaManager` and `SagaRepository`. The `SagaRepository` will use a `SagaStore` available in the context \(defaulting to `JPASagaStore` if JPA is found\) for the actual storage of sagas.

To use different `SagaStore`s for sagas, provide the bean name of the `SagaStore` to use in the `sagaStore` attribute of each `@Saga` annotation.

Sagas will have resources injected from the application context. Note that this does not mean Spring-injecting is used to inject these resources. The `@Autowired` and `@javax.inject.Inject` annotation can be used to demarcate dependencies, but they are injected by Axon by looking for these annotations on fields and methods. Constructor injection is not \(yet\) supported.

To tune the configuration of sagas, it is possible to define a custom `SagaConfiguration` bean. For an annotated saga class, Axon will attempt to find a configuration for that saga. It does so by checking for a bean of type `SagaConfiguration` with a specific name. For a saga class called `MySaga`, the bean that Axon looks for is `mySagaConfiguration`. If no such bean is found, it creates a configuration based on available components.

If a `SagaConfiguration` instance is present for an annotated saga, that configuration will be used to retrieve and register components for sagas of that type. If the `SagaConfiguration` bean is not named as described above, it is possible that saga will be registered twice. It will then receive events in duplicate. To prevent this, you can specify the bean name of the `SagaConfiguration` using the `@Saga` annotation:

```java
@Autowired
public void configureSagaEventProcessing(EventProcessingConfigurer config) {
    config.registerTokenStore("MySagaProcessor", c -> new MySagaCustomTokenStore())
}
```
{% endtab %}
{% endtabs %}

## Error Handling

Errors are inevitable. Depending on where they happen, you may want to respond differently.

By default exceptions raised by event handlers are logged and processing continues with the next events. When an exception is thrown when a processor is trying to commit a transaction, update a token, or in any other other part of the process, the exception will be propagated. In case of a Tracking Event Processor, this means the processor will go into error mode, releasing any tokens and retrying at an incremental interval \(starting at 1 second, up to max 60 seconds\). A Subscribing Event Processor will report a publication error to the component that provided the event.

To change this behavior, there are two levels at which you can customize how Axon deals with exceptions:

### Exceptions raised by event handler methods

By default these exceptions are logged and processing continues with the next handler or message.

This behavior can be configured per processing group:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
eventProcessingConfigurer.registerDefaultListenerInvocationErrorHandler(conf -> /* create error handler */);

// or for a specific processing group:
eventProcessingConfigurer.registerListenerInvocationErrorHandler("processingGroup", conf -> /* create error handler */);
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Autowired
public void configure(EventProcessingConfigurer config) {
    config.registerDefaultListenerInvocationErrorHandler(conf -> /* create error handler */);

    // or for a specific processing group:
    config.registerListenerInvocationErrorHandler("processingGroup", conf -> /* create error handler */);
}
```
{% endtab %}
{% endtabs %}

It is easy to implement custom error handling behavior. The error handling method to implement provides the exception, the event that was handled, and a reference to the handler that was handling the message. You can choose to retry, ignore or rethrow the exception. In the latter case, the exception will bubble up to the event processor level.

### Exceptions during processing

Exceptions that occur outside of the scope of an event handler, or have bubbled up from there, are handled by the ErrorHandler. The default behavior depends on the processor implementation:

A `TrackingEventProcessor` will go into Error Mode. Then, it will retry processing the event using an incremental back-off period. It will start at 1 second and double after each attempt until a maximum wait time of 60 seconds per attempt is achieved. This back-off time ensures that if another node is able to process events successfully, it will have the opportunity to claim the token required to process the event.

The `SubscribingEventProcessor` will have the exception bubble up to the publishing component of the event, allowing it to deal with it, accordingly.

To customize the behavior, you can configure an error handler on the event processor level.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
eventProcessingConfigurer.registerDefaultErrorHandler(conf -> /* create error handler */);

// or for a specific processor:
eventProcessingConfigurer.registerErrorHandler("processorName", conf -> /* create error handler */);
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Autowired
public void configure(EventProcessingConfigurer config) {
    config.registerDefaultErrorHandler(conf -> /* create error handler */);

    // or for a specific processing group:
    config.registerErrorHandler("processingGroup", conf -> /* create error handler */);
}
```
{% endtab %}
{% endtabs %}

To implement custom behavior, implement the `ErrorHandler`'s single method. Based on the provided `ErrorContext` object, you can decide to ignore the error, schedule retries, perform dead-letter-queue delivery or rethrow the exception.

## Token Store

Tracking event processors, unlike subscribing ones, need a token store to store their progress in. Each message a tracking processor receives through its event stream is accompanied by a token. This token allows the processor to reopen the stream at any later point, picking up where it left off with the last event.

The Configuration API takes the token store, as well as most other components processors need from the global configuration instance. If no token store is explicitly defined, an `InMemoryTokenStore` is used, which is _not_ recommended in production.

{% tabs %}
{% tab title="Axon Configuration API" %}
To configure a token store, use the `EventProcessingConfigurer` to define which implementation to use.

To configure a default TokenStore for all processors:

```java
 Configurer.eventProcessing().registerTokenStore(conf -> ... create token store ...)
```

Alternatively, to configure a TokenStore for a specific processor, use:

```java
Configurer.eventProcessing().registerTokenStore("processorName", conf -> ... create token store ...)`.
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
The default TokenStore implementation is defined based on dependencies available in Spring Boot, in the following order:

1. If any `TokenStore` bean is defined, that bean is used
2. Otherwise, if an `EntityManager` is available, the `JpaTokenStore` is defined.
3. Otherwise, if a `DataSource` is defined, the `JdbcTokenStore` is created
4. Lastly, the `InMemoryToken` store is used

To override the TokenStore, either define a bean in a Spring `@Configuration` class:

```java
@Bean
public TokenStore myCustomTokenStore() {
    return new MyCustomTokenStore();
}
```

Alternatively, inject the `EventProcessingConfigurer`, which allows more fine-grained customization:

```java
@Autowired
public void configure(EventProcessingConfigurer epConfig) {
    epConfig.registerTokenStore(conf -> new MyCustomTokenStore());

    // or, to define one for a single processor:
    epConfig.registerTokenStore("processorName", conf -> new MyCustomTokenStore());
}
```
{% endtab %}
{% endtabs %}

Note that you can override the token store to use with tracking processors in the `EventProcessingConfiguration` that defines that processor. Where possible, it is recommended to use a token store that stores tokens in the same database as where the event handlers update the view models. This way, changes to the view model can be stored atomically with the changed tokens. This guarantees exactly once processing semantics.

## Splitting and Merging Tracking Tokens

It is possible to tune the performance of Tracking Event Processors by increasing the number of threads processing events on high load by splitting segments and reducing the number of threads when load reduces by merging segments.  
Splitting and merging is allowed at runtime which allows you to dynamically control the number of segments. This can be done through the Axon Server API or through Axon Framework using the methods `splitSegment(int segmentId)` and `mergeSegment(int segmentId)` from `TrackingEventProcessor` by providing the segmentId of the segment you want to split or merge.

> **Segment Selection Considerations**
>
> By splitting/merging using Axon Server the most appropriate segment to split or merge is chosen for you. When using the Axon Framework API directly, the segment to split/merge should be deduced by the developer themselves:
>
> * Split: for fair balancing, a split is ideally performed on the biggest segment
> * Merge: for fair balancing, a merge is ideally performed on the smallest segment

## Parallel processing

Tracking processors can use multiple threads to process an event stream. They do so by claiming a segment which is identified by a number. Normally, a single thread will process a single segment.

The number of segments used can be defined. When an event processor starts for the first time, it can initialize a number of segments. This number defines the maximum number of threads that can process events simultaneously. Each node running of a tracking event processor will attempt to start its configured amount of threads to start processing events.

Event handlers may have specific expectations on the ordering of events. If this is the case, the processor must ensure these events are sent to these handlers in that specific order. Axon uses the `SequencingPolicy` for this. The `SequencingPolicy` is a function that returns a value for any given message. If the return value of the `SequencingPolicy` function is equal for two distinct event messages, it means that those messages must be processed sequentially. By default, Axon components will use the `SequentialPerAggregatePolicy`, which makes it so that events published by the same aggregate instance will be handled sequentially.

A saga instance is never invoked concurrently by multiple threads. Therefore, a sequencing policy for a saga is irrelevant. Axon will ensure each saga instance receives the events it needs to process in the order they have been published on the event bus.

> **Parallel processing and Subscribing Event Processors**
>
> Note that subscribing event processors don't manage their own threads. 
> Therefore, it is not possible to configure how they should receive their events. 
> Effectively, they will always work on a sequential-per-aggregate basis, as that is generally the level of concurrency in the command handling component.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
DefaultConfigurer.defaultConfiguration()
    .eventProcessing(eventProcessingConfigurer -> eventProcessingConfigurer.registerTrackingEventProcessor(
            "myProcessor", 
            c -> c.eventStore(), 
            c -> c.getComponent(
                TrackingEventProcessorConfiguration.class, 
                () -> TrackingEventProcessorConfiguration.forParallelProcessing(3)
            )
        )
    );
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
You can configure the number of threads \(on this instance\) as well as the initial number of segments that a processor should define, if none are yet available.

```text
axon.eventhandling.processors.name.mode=tracking
# Sets the number of maximum number threads to start on this node
axon.eventhandling.processors.name.threadCount=2
# Sets the initial number of segments (i.e. defines the maximum number of overall threads)
axon.eventhandling.processors.name.initialSegmentCount=4
```
{% endtab %}
{% endtabs %}

### Sequential processing

Even though events are processed asynchronously from their publisher, it is often desirable to process certain events in the order they are published. In Axon this is controlled by the `SequencingPolicy`.

The `SequencingPolicy` defines whether events must be handled sequentially, in parallel or a combination of both. Policies return a sequence identifier of a given event. If the policy returns an equal identifier for two events, this means that they must be handled sequentially by the event handler. A `null` sequence identifier means the event may be processed in parallel with any other event.

Axon provides a number of common policies you can use:

* The `FullConcurrencyPolicy` will tell Axon that this event handler may handle all events concurrently.
  This means that there is no relationship between the events that require them to be processed in a particular order.

* The `SequentialPolicy` tells Axon that all events must be processed sequentially.
  Handling of an event will start when the handling of a previous event has finished.

* The default policy is the `SequentialPerAggregatePolicy`. 
  It will force domain events that were raised from the same aggregate to be handled sequentially.
  However, events from different aggregates may be handled concurrently.
  This is typically a suitable policy to use for event listeners that update details from aggregates in database tables.

Besides these provided policies, you can define your own. All policies must implement the `SequencingPolicy` interface. This interface defines a single method, `getSequenceIdentifierFor`, that returns the sequence identifier for a given event. Events for which an equal sequence identifier is returned must be processed sequentially. Events that produce a different sequence identifier may be processed concurrently. Policy implementations may return `null` if the event may be processed in parallel to any other event.

```java
eventProcesingConfigurer.registerSequencingPolicy("processingGroup", conf -> /* define policy */);

// or, to change the default:

eventProcesingConfigurer.registerDefaultSequencingPolicy(conf -> /* define policy */);
```

### Multi-node processing

For tracking processors, it doesn't matter whether the threads handling the events are all running on the same node or on different nodes hosting the same \(logical\) tracking processor. When two instances of a tracking processor with the same name are active on different machines, they are considered two instances of the same logical processor. They will 'compete' for segments of the event stream. Each instance will 'claim' a segment, preventing events assigned to that segment from being processed on the other nodes.

The `TokenStore` instance will use the JVM's name \(usually a combination of the host name and process ID\) as the default `nodeId`. This can be overridden in `TokenStore` implementations that support multi-node processing.

## Distributing Events

Distributing events in inter-process environments is supported with Axon Server by default.

Alternatively, you can choose other components that you can find in one of the extension modules \(Spring AMQP, Kafka\).

## Replaying events

In cases when you want to rebuild projections \(view models\), replaying past events comes in handy. 
The idea is to start from the beginning of time and invoke all event handlers anew. 
The `TrackingEventProcessor` supports replaying of events. 
In order to achieve that, you should invoke the `resetTokens()` method on it. 

It is important to know that the tracking event processor must not be in active state when starting a reset. 
Hence it is required to shut it down first, then reset it and once this was successful, after which it can be started up again.

Initiating a replay through the `TrackingEventProcessor` opens up an API to tap into the process of replaying. 
It is for example possible to define a `@ResetHandler`, so you can do some preparation prior to resetting.
 
Let's take a look at how we can accomplish a replay of a tracking event processor. 
First, we will see one simple projecting class:

```java
@AllowReplay // 1.
@ProcessingGroup("card-summary")
public class CardSummaryProjection {
    //...
    @EventHandler
    @DisallowReplay // 2. - It is possible to prevent some handlers from being replayed
    public void on(CardIssuedEvent event) {
        // This event handler performs a "side effect",
        //  like sending an e-mail or a sms.
        // Neither, is something we want to reoccur when a 
        //  replay happens, hence we disallow this method 
        //  to be replayed
    }

    @EventHandler
    public void on(CardRedeemedEvent event, ReplayStatus replayStatus /* 3. */) {
        // We can wire a ReplayStatus here so we can see whether this
        // event is delivered to our handler as a 'REGULAR' event or
        // a 'REPLAY' event
        // Perform event handling
    }    

    @ResetHandler // 4. - This method will be called before replay starts
    public void onReset(ResetContext resetContext) {
        // Do pre-reset logic, like clearing out the projection table for a
        // clean slate. The given resetContext is [optional], allowing the 
        // user to specify in what context a reset was executed.
    }
    //...
}
```

The `CardSummaryProjection` shows a couple of interesting things to take note of when it comes to "being aware" of a reset is in progress:

1. An `@AllowReplay` can be used, situated either on an entire class or an `@EventHandler` annotated method. It defines whether the given class or method should be invoked when a replay is in transit.
2. Next to allowing a replay, `@DisallowReplay` could also be used. Similar to `@AllowReplay` it can be placed on class level and on methods, where it serves the purpose of defining whether the class / method should **not** be invoked when a replay is in transit.   
3. To have more fine-grained control on what (not) to do during a replay, the `ReplayStatus` parameter can be added. It is an additional parameter which can be added to `@EventHandler` annotated methods, allowing conditional operations within a method to be performed based on whether a replay is in transit yes or no.
4. If there is certain pre-reset logic which should be performed, like clearing out a projection table, the `@ResetHandler` annotation should be used. This annotation can only be placed on a method, allowing the addition of a reset context if necessary. The `resetContext` passed along in the `@ResetHandler` originates from the operation initiating the `TrackingEventProcessor#resetTokens(R resetContext)` method. The type of the `resetContext` is up to the user. 

With all this in place, we are ready to initiate a reset from our `TrackingEventProcessor`.
To that end, we need to have access to the `TrackingEventProcessor` we want to reset.
For this you should retrieve the `EventProcessingConfiguration` available in the main `Configuration`.
Added, this is where we can provide an optional reset context to be passed along in the `@ResetHandler`:

{% tabs %}
{% tab title="Reset without reset context" %}
```java
public class ResetService {
    //...
    public void reset(Configuration config) {
        EventProcessingConfiguration eventProcessingConfig = config.eventProcessingConfiguration();
        eventProcessingConfig.eventProcessor("card-summary", TrackingEventProcessor.class)
                             .ifPresent(processor -> {
                                 processor.shutDown();
                                 processor.resetTokens();
                                 processor.start();
                             });
    }
}
```
{% endtab %}

{% tab title="Reset with reset context" %}
```java
public class ResetService {
    //...
    public <R> void resetWithContext(Configuration config, R resetContext) {
        EventProcessingConfiguration eventProcessingConfig = config.eventProcessingConfiguration();
        eventProcessingConfig.eventProcessor("card-summary", TrackingEventProcessor.class)
                             .ifPresent(processor -> {
                                 processor.shutDown();
                                 processor.resetTokens(resetContext);
                                 processor.start();
                             });
    }
}
```
{% endtab %}
{% endtabs %}

It is possible to provide a change listener which can validate whenever the replay is done.
More specifically, a `EventTrackerStatusChangeListener` can be configured through the `TrackingEventProcessorConfiguration`.
See the [monitoring and metrics](../monitoring-and-metrics.md#event-tracker-status-a-idevent-tracker-statusa) for more specifics on the change listener.

> **Partial Replays**
>
> It is possible to provide a token position to be used when resetting a `TrackingEventProcessor`, thus specifying from which point in the event log it should start replaying the events.
> This would require the usage of the `TrackingEventProcessor#resetTokens(TrackingToken)` or `TrackingEventProcessor#resetTokens(Function<StreamableMessageSource<TrackedEventMessage<?>>, TrackingToken>)` method, which both provide the `TrackingToken` from which the reset should start.
>
> How to customize a tracking token position is described [here](event-processors.md#custom-tracking-token-position).

## Custom tracking token position

Prior to Axon release 3.3, you could only reset a `TrackingEventProcessor` to the beginning of the event stream. As of version 3.3 functionality for starting a `TrackingEventProcessor` from a custom position has been introduced. The `TrackingEventProcessorConfiguration` provides the option to set an initial token for a given `TrackingEventProcessor` through the `andInitialTrackingToken(Function<StreamableMessageSource, TrackingToken>)` builder method. As an input parameter for the token builder function, we receive a `StreamableMessageSource` which gives us three possibilities to build a token:

* From the head of event stream: `createHeadToken()`.
* From the tail of event stream: `createTailToken()`.
* From some point in time: `createTokenAt(Instant)` and `createTokenSince(duration)` -

  Creates a token that tracks all events after given time.

  If there is an event exactly at the given time, it will be taken into account too.

Of course, you can completely disregard the `StreamableMessageSource` input parameter and create a token by yourself.

Below we can see an example of creating a `TrackingEventProcessorConfiguration` with an initial token on `"2007-12-03T10:15:30.00Z"`:

```java
public class Configuration {

    public TrackingEventProcessorConfiguration customConfiguration() {
        return TrackingEventProcessorConfiguration
                .forSingleThreadedProcessing()
                .andInitialTrackingToken(streamableMessageSource -> streamableMessageSource.createTokenAt(
                        Instant.parse("2007-12-03T10:15:30.00Z")
                ));
    }
}
```

