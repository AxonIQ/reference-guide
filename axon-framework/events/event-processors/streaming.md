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

The number of segments used can be defined. When an event processor starts for the first time, it can initialize a number of segments. This number defines the maximum number of threads that can process events simultaneously.

Each node running of a tracking event processor will attempt to start its configured amount of threads to start processing events. The TrackingEventProcessor can claim a single segment per thread, if the configured number of threads are less than number of configured segments, some of the events might not be handled. Hence it is recommended to have the number of threads on every node more than or equal to the total number of segments.

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
# Sets the maximum number threads to start on this node
axon.eventhandling.processors.name.threadCount=5
# Sets the initial number of segments
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

## Replaying events

When you want to rebuild projections \(view models\), replaying past events comes in handy.
The idea is to start from the beginning of time and invoke all event handlers anew.
The `TrackingEventProcessor` supports replaying of events.
In order to achieve that, you should invoke the `resetTokens()` method on it.

It is important to know that the tracking event processor must not be in active state when starting a reset.
Hence it is required to be shut down first, then reset, and once successful, it can be started up again.

Initiating a replay through the `TrackingEventProcessor` opens up an API to tap into the process of replaying.
It is, for example, possible to define a `@ResetHandler`, so you can do some preparations prior to resetting.

Let's take a look at how we can accomplish a replay of a tracking event processor.
First, let's take a look at one simple projecting class:

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
2. In addition to allowing a replay, `@DisallowReplay` can also be used. Similarly to `@AllowReplay` it can be placed on class level and on methods, where it serves the purpose of defining whether the class / method should **not** be invoked when a replay is in transit.
3. To have more fine-grained control on what (not) to do during a replay, the `ReplayStatus` parameter can be added. It is an additional parameter which can be added to `@EventHandler` annotated methods, allowing conditional operations within a method to be performed based on whether a replay is in transit or not.
4. If there is a certain pre-reset logic that should be performed, such as clearing out a projection table, the `@ResetHandler` annotation should be used. This annotation can only be placed on a method, allowing the addition of a reset context, if necessary. The `resetContext` passed along in the `@ResetHandler` originates from the operation, initiating the `TrackingEventProcessor#resetTokens(R resetContext)` method. The type of the `resetContext` is up to the user.

With all this in place, we are ready to initiate a reset from our `TrackingEventProcessor`.
To that end, we need to have access to the `TrackingEventProcessor` we want to reset.
For this you should retrieve the `EventProcessingConfiguration` available in the main `Configuration`.
Additionally, this is where we can provide an optional reset context to be passed along in the `@ResetHandler`:

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
More specifically, an `EventTrackerStatusChangeListener` can be configured through the `TrackingEventProcessorConfiguration`.
See the [monitoring and metrics](../../monitoring-and-metrics.md#event-tracker-status-a-idevent-tracker-statusa) for more specifics on the change listener.

> **Partial Replays**
>
> It is possible to provide a token position to be used when resetting a `TrackingEventProcessor`; thus, specifying from which point in the event log it should start replaying the events.
> This would require the usage of the `TrackingEventProcessor#resetTokens(TrackingToken)` or `TrackingEventProcessor#resetTokens(Function<StreamableMessageSource<TrackedEventMessage<?>>, TrackingToken>)` method, which both provide the `TrackingToken` from which the reset should start.
>
> How to customize a tracking token position is described [here](streaming.md#custom-tracking-token-position).

## Custom tracking token position

Prior to Axon release 3.3, you could only reset a `TrackingEventProcessor` to the beginning of the event stream. As of version 3.3 functionality for starting a `TrackingEventProcessor` from a custom position has been introduced. The `TrackingEventProcessorConfiguration` provides the option to set an initial token for a given `TrackingEventProcessor` through the `andInitialTrackingToken(Function<StreamableMessageSource, TrackingToken>)` builder method. As an input parameter for the token builder function, we receive a `StreamableMessageSource` which gives us three possibilities to build a token:

* From the head of event stream: `createHeadToken()`.
* From the tail of event stream: `createTailToken()`.
* From some point in time: `createTokenAt(Instant)` and `createTokenSince(duration)` -

  Creates a token that tracks all events after a given time.

  If there is an event exactly at that given moment in time, that event will also be taken into account.

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

## Pooled Streaming Event Processor

Next to the `TrackingEventProcessor`, there is another `EventProcessor` implementation which provides the same operations with a different processing approach behind it.
This is the so-called `PooledStreamingEventProcessor`.

It allows for [splitting, merging](#splitting-and-merging-tracking-tokens) and [resetting](#replaying-events) the segments of a processor, just as with the `TrackingEventProcessor`.
Under the hoods, it however uses two threads pools, instead of the single fixed set of threads used by the `TrackingEventProcessor`.
The first thread pool is in charge of open a stream with the event source and to delegate all the work (e.g. which events to handle, split/merge, etc.).
The second thread pool is dealing with all the segments the `PooledStreamingEventProcessor` could claim.

When it comes to configuring the `PooledStreamingEventProcessor`, you can take the following approaches:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class Configuration {
    
    public void configurePooledProcessor(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer
            // Defaults all Event Processors to PooledStreamingEventProcessors instances
            .usingPooledStreamingEventProcessors()
            // Configures the "processing-group" to be a PooledStreamingEventProcessors
            .registerPooledStreamingEventProcessor("processing-group")
            // Configures the "processing-group" to be a PooledStreamingEventProcessors, using the EventStore as a message source
            .registerPooledStreamingEventProcessor("processing-group", config -> config.eventStore())
            // The same "processing-group"  configuration, including a PooledStreamingProcessorConfiguration allowing complete configuration of the PooledStreamingEventProcessors 
            .registerPooledStreamingEventProcessor(
                "processing-group", config -> config.eventStore(),
                (config, builder) -> builder /* Invoke the 'builder's methods for futher configuraiton'*/
            );  
    }
}
```
{% endtab %}
{% tab title="Spring Boot AutoConfiguration" %}
```text
# Defines that a processor "processing-group" should be a PooledStreamingEventProcessor
axon.eventhandling.processors.processing-group.mode=pooled
# Sets the event batch size for "processing-group" to 1024
axon.eventhandling.processors.processing-group.batchSize=1024
# Sets the initial number of segments for "processing-group" to 32
axon.eventhandling.processors.processing-group.initialSegmentCount=32
```
{% endtab %}
{% endtabs %}