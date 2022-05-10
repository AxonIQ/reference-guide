# Streaming Event Processor

The `StreamingEventProcessor`, or Streaming Processor for short, is a type of [Event Processor](README.md).
As any Event Processor, it serves as the technical aspect to handle events by invoking the event handlers written in an Axon application.

The Streaming Processor defines itself by receiving the events from a `StreamableMessageSource`.
The `StreamableMessageSource` is an infrastructure component through which we can open a stream of events.
The source can also specify positions on the event stream, so-called [Tracking Tokens](#tracking-tokens), used as start positions when opening an event stream.
An example of a `StreamableMessageSource` is the [`EventStore`](../event-bus-and-event-store.md#event-store), like for example [Axon Server](../../../axon-server-introduction.md) or an [RDBMS](../event-bus-and-event-store.md#embedded-event-store).

Furthermore, Streaming Processors use separate threads to process the events retrieved from the `StreamableMessageSource`.
Using separate threads decouples the `StreamingEventProcessor` from other operations (e.g., event publication or command handling), allowing for cleaner separation within any application.  
Using separate threads allows for [parallelization](#parallel-processing) of the event load, either within a single JVM or between several.

When starting a Streaming Processor, it will open an event stream through the configured `StreamableMessageSource`. The first time a stream has started, it, by default, will begin at the tail (the oldest/the very first token) of the stream.
It keeps track of the event processing progress while traversing the stream.
It does so by storing the Tracking Tokens, or _tokens_ for short, accompanying the events.
This solution works towards tracking the progress since the tokens specify the event's position on the stream.

> **Head or Tail?**
> 
> The oldest (very first) token is located at the tail of the stream, and the latest (newest) token is positioned at the head of the stream. 

Maintaining the progress through tokens makes a Streaming Processor
1. able to deal with stopping and starting the processor,
2. more resilient against unintended shutdowns, and
3. the token provides a means to [replay](#replaying-events) events by adjusting the position of tokens.

All combined, the Streaming Processor allows for decoupling, parallelization, resiliency, and replay-ability.
It is these features that make the Streaming Processor the logical choice for the majority of applications.
Due to this, the "Tracking Event Processor," a type of Streaming Processor, is the default Event Processor.

> **Default Event Processor**
>
> Which `EventProcessor` type becomes the default processor depends on the event message source available in your application.
> In the majority of use cases, an Event Store is present.
> As the Event Store is a type of `StreamableMessageSource`, the default will switch to the Tracking Event Processor.
> 
> If the application only has an Event Bus configured, the framework will lack a `StreamableMessageSource`.
> It will fall back to the [Subscribing Event Processor](subscribing.md) as the default in these scenarios.
> This implementation will use the configured `EventBus` as its `SubscribableMessageSource`.

There are two implementations of Streaming Processor available in Axon Framework:

1. the Tracking Event Processor (TEP for short), and
2. the Pooled Streaming Event Processor (PSEP for short).

Both implementations support the same set of operations. 
Operations like replaying events through a [reset](#replaying-events), [parallelism](#parallel-processing) and tracking the progress with [tokens](#tracking-tokens).
They diverge on their threading approach and work separation, as discussed in more detail in [this](#thread-configuration) section.

## Configuring

The Streaming Processors have several additional components that you can configure, next to the [base options](README.md#general-processor-configuration).
For other streaming processor features that are configurable, we refer to their respective sections for more details.
This chapter will cover how to configure a [Tracking](#configuring-a-tracking-processor) or [Pooled Streaming](#configuring-a-pooled-streaming-processor) Processor respectively.

### Configuring a Tracking Processor

Firstly, to specify that new event processors should default to a `TrackingEventProcessor`, you can invoke the `usingTrackingEventProcessors` method:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig { 
    // ...
    public void configureProcessorDefault(EventProcessingConfigurer processingConfigurer) { 
        processingConfigurer.usingTrackingEventProcessors();  
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureProcessorDefault(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer.usingTrackingEventProcessors();
    }
}
```
{% endtab %}
{% endtabs %}

For a specific Event Processor to be a Tracking instance, `registerTrackingEventProcessor` is used:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureTrackingProcessors(EventProcessingConfigurer processingConfigurer) {
        // This configuration object allows for fine-grained control over the Tracking Processor
        TrackingEventProcessorConfiguration tepConfig =
              TrackingEventProcessorConfiguration.forSingleThreadedProcessing();
        
        // To configure a processor to be tracking ...
        processingConfigurer.registerTrackingEventProcessor("my-processor")
                            // ... to define a specific StreamableMessageSource ... 
                            .registerTrackingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */
                            )
                            // ... to provide additional configuration ...
                            .registerTrackingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */,
                                    conf -> tepConfig
                            );
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Java" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureTrackingProcessors(EventProcessingConfigurer processingConfigurer) {
        // This configuration object allows for fine-grained control over the Tracking Processor
        TrackingEventProcessorConfiguration tepConfig =
              TrackingEventProcessorConfiguration.forSingleThreadedProcessing();
        
        // To configure a processor to be tracking ...
        processingConfigurer.registerTrackingEventProcessor("my-processor")
                            // ... to define a specific StreamableMessageSource ... 
                            .registerTrackingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */
                            )
                            // ... to provide additional configuration ...
                            .registerTrackingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */, 
                                    conf -> tepConfig
                            );
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties file" %}
A properties file allows the configuration of some fields on an Event Processor.
Do note that the Java configuration provides more degrees of freedom.

```text
axon.eventhandling.processors.my-processor.mode=tracking
axon.eventhandling.processors.my-processor.source=eventStore
```

If the name of an event processor contains periods `.`, use the map notation:

```text
axon.eventhandling.processors[my.processor].mode=tracking
axon.eventhandling.processors[my.processor].source=eventStore
```
{% endtab %}
{% endtabs %}

For more fine-grained control when configuring a Tracking Processor, the `TrackingEventProcessorConfiguration` can be used.
When invoking the `registerTrackingEventProcessor` method, you can provide a tracking processor configuration object, or you can register the configuration instance explicitly:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void registerTrackingProcessorConfig(EventProcessingConfigurer processingConfigurer) {
        TrackingEventProcessorConfiguration tepConfig =
                TrackingEventProcessorConfiguration.forSingleThreadedProcessing();
            
        // To register a default tracking config ...
        processingConfigurer.registerTrackingEventProcessorConfiguration(config -> tepConfig)
                            // ... to register a config for a specific processor.
                            .registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Java" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void registerTrackingProcessorConfig(EventProcessingConfigurer processingConfigurer) {
        TrackingEventProcessorConfiguration tepConfig =
                  TrackingEventProcessorConfiguration.forSingleThreadedProcessing();
    
        // To register a default tracking config ...
        processingConfigurer.registerTrackingEventProcessorConfiguration(config -> tepConfig)
                            // ... to register a config for a specific processor.
                            .registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}
{% endtabs %}

### Configuring a Pooled Streaming Processor

Firstly, to specify that every new processors should default to a `PooledStreamingEventProcessor`, you can invoke the `usingPooledStreamingProcessors` method:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig { 
    // ...
    public void configureProcessorDefault(EventProcessingConfigurer processingConfigurer) { 
        processingConfigurer.usingPooledStreamingProcessors();  
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureProcessorDefault(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer.usingPooledStreamingProcessors();
    }
}
```
{% endtab %}
{% endtabs %}

For a specific Event Processor to be a Pooled Streaming instance, `registerPooledStreamingProcessor` is used:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configurePooledStreamingProcessors(EventProcessingConfigurer processingConfigurer) {
          // This configuration object allows for fine-grained control over the Pooled Streaming Processor
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig = 
                (config, builder) -> builder/* ... */;
          
        // To configure a processor to be pooled streaming ...
        processingConfigurer.registerPooledStreamingEventProcessor("my-processor")
                            // ... to define a specific StreamableMessageSource ... 
                            .registerPooledStreamingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */
                            )
                            // ... to provide additional configuration ...
                            .registerPooledStreamingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */, psepConfig
                            );
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Java" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configurePooledStreamingProcessors(EventProcessingConfigurer processingConfigurer) {
        // This configuration object allows for fine-grained control over the Pooled Streaming Processor
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig =
                (config, builder) -> builder/* ... */;
    
        // To configure a processor to be pooled streaming ...
        processingConfigurer.registerPooledStreamingEventProcessor("my-processor")
                            // ... to define a specific StreamableMessageSource ... 
                            .registerPooledStreamingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */
                            )
                            // ... to provide additional configuration ...
                            .registerPooledStreamingEventProcessor(
                                    "my-processor", conf -> /* create/return StreamableMessageSource */, psepConfig
                            );
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties file" %}
A properties file allows the configuration of some fields on an Event Processor.
Do note that the Java configuration provides more degrees of freedom.

```text
axon.eventhandling.processors.my-processor.mode=pooled
axon.eventhandling.processors.my-processor.source=eventStore
```

If the name of an event processor contains periods `.`, use the map notation:

```text
axon.eventhandling.processors[my.processor].mode=pooled
axon.eventhandling.processors[my.processor].source=eventStore
```
{% endtab %}
{% endtabs %}

For more fine-grained control when configuring a Pooled Streaming Processor, the `PooledStreamingProcessorConfiguration` can be used.
When invoking the `registerPooledStreamingEventProcessor` method, you can provide a pooled streaming processor configuration object, or you can register the configuration instance explicitly:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void registerPooledStreamingProcessorConfig(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig = 
                (config, builder) -> builder/* ... */;
          
        // To register a default pooled streaming config ...
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration(psepConfig)
                            // ... to register a config for a specific processor.
                            .registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Java" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void registerPooledStreamingProcessorConfig(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig =
                (config, builder) -> builder/* ... */;
    
        // To register a default pooled streaming config ...
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration(psepConfig)
                            // ... to register a config for a specific processor.
                            .registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}
{% endtabs %}

## Error Mode

The error mode differs between the Tracking- and Pooled Streaming Event Processor.

Whenever the [error handler](README.md#event-processor-error-handler) rethrows an exception, a `TrackingEventProcessor` will retry processing the event using an incremental back-off period.
It will start at 1 second and double after each attempt until it reaches the maximum wait time of 60 seconds per attempt.
This back-off time ensures that in a distributed environment, when another node is able to process events, it will have the opportunity to claim the [token](#tracking-tokens) required to process the event.

The `PooledStreamingEventProcessor` simply aborts the failed part of the process.
The Pooled Streaming Processor can deal with this since the [threading mode](#pooled-streaming-processor-threading) is different from the Tracking Processor.
As such, the chance is high the failed process will be picked up quickly by another thread within the same JVM.
This chance increases further whenever the PSEP instance is distributed over several application instances.

## Tracking Tokens

A vital attribute of the Streaming Event Processor is its capability to keep and maintain the processing progress.
It does so through the `TrackingToken`, the "token" for short.
Such a token accompanies each message a streaming processor receives through its event stream.
It's this token that:

1. specifies the position of the event on the overall stream, and
2. is used by the Streaming Processor to open the event stream at the desired position on start-up.

Using tokens gives the Streaming Event Processor several benefits, like:

* Being able to reopen the stream at any later point, picking up where it left off with the last event.
* Dealing with unintended shutdowns without losing track of the last events they've handled.
* Collaboration over the event handling load from two perspectives. 
  First, the tokens make sure only a single thread is actively processing specific events. 
  Secondly, it allows [parallelization](#parallel-processing) of the load over several threads or nodes of a Streaming Processor.
* [Replaying](#replaying-events) events by adjusting the token position of that processor.

To be able to reopen the stream at a later point, we should keep the progress somewhere.
The progress is kept by updating and saving the `TrackingToken` after handling batches of events.
Keeping the progress requires CRUD operation, for which the Streaming Processor uses the [`TokenStore`](#token-store).

For a Streaming Processor to process any events, it needs "a claim" on a `TrackingToken`.
The processor will update this claim every time it has finished handling a batch of events.
This so-called "claim extension" is, just as updating and saving of tokens, delegated to the Token Store.
Hence, the Streaming Processors achieves collaboration among instances/threads through token claims.

In the absence of a claim, a processor will actively try to retrieve one.
If a token claim is not extended for a configurable time window, other processor threads are able to "steal" the claim.
Token stealing can, for example, happen if event processing is slow or encountered some exceptions.
 
### Initial Tracking Token

The Streaming Processor uses a `StreamableMessageSource` to retrieve a stream of events that will open on start-up.
It requires a `TrackingToken` to open this stream, which it will fetch from the `TokenStore`.
However, if a Streaming Processor starts for the first time, there is no `TrackingToken` present to open the stream with yet.

Whenever this situation occurs, a Streaming Processor will construct an "initial token."
By default, the initial token will start at the tail of the event stream.
Thus, the processor will begin at the start and handle every event present in the message source.
This start position is configurable, as is described [here](#token-configuration).

> **A Saga's Streaming Processor initial position**
>
> A Streaming Processor dedicated to a [Saga](../../sagas/README.md) will default the initial token to the head of the stream.
> The default initial token position ensures that the Saga does not react to events from the past, as in most cases, this would introduce unwanted side effects.

Conceptually there are a couple of scenarios when a processor builds an initial token on application startup.
The obvious one is already shared, namely when a processor starts for the first time.
There are, however, also other situations when a token is built that might be unexpected, like:

* The `TokenStore` has (accidentally) been cleared between application runs, thus losing the stored tokens.
* The application running the processor starts in a new environment (e.g., test or acceptance) for the first time.
* An `InMemoryTokenStore` was used, and hence the processor could never persist the token to begin with.
* The application is (accidentally) pointing to another storage solution than expected.

Whenever a Streaming Processor's event handlers show unexpected behavior in the form of missed or reprocessed events, a new initial token might have been triggered.
In those cases, we recommend to validate if any of the above situations occurred.

### Token Configuration

There are a couple of things we can configure when it comes to tokens.
We can separate these options in "initial token" and "token claim" configuration, as described in the following sections:

**Initial Token**

The [initial token](#initial-tracking-token) for a `StreamingEventProcessor` is configurable for every processor instance.
When configuring the initial token builder function, the received input parameter is the `StreamableMessageSource`.
The message source, in turn, gives three possibilities to build a token, namely:

1. `createHeadToken()` - Creates a token from the head of the event stream. 
2. `createTailToken()` - Creates a token from the tail of the event stream. Creating tail tokens is the default value for most Streaming Processors.
3. `createTokenAt(Instant)` / `createTokenSince(Duration)` - Creates a token that tracks all events after a given time.
   If there is an event precisely at that given moment in time, it will also be taken into account.

Of course, you can completely disregard the `StreamableMessageSource` input parameter and create a token by yourself.
Consider the following snippets if you want to configure a different initial token:

{% tabs %}
{% tab title="Tracking Processor - Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureInitialTrackingToken(EventProcessingConfigurer processingConfigurer) {
        TrackingEventProcessorConfiguration tepConfig = 
                TrackingEventProcessorConfiguration.forSingleThreadedProcessing()
                                                   .andInitialTrackingToken(StreamableMessageSource::createHeadToken);
        
        processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Tracking Processor - Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureInitialTrackingToken(EventProcessingConfigurer processingConfigurer) {
          TrackingEventProcessorConfiguration tepConfig = 
                  TrackingEventProcessorConfiguration.forSingleThreadedProcessing()
                                                     .andInitialTrackingToken(StreamableMessageSource::createTailToken);
          
          processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureInitialTrackingToken(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig = 
                (config, builder) -> builder.initialToken(messageSource -> messageSource.createTokenSince(
                        messageSource -> messageSource.createTokenAt(Instant.parse("20020-12-01T10:15:30.00Z"))
                ));
        
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureInitialTrackingToken(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig = 
                (config, builder) -> builder.initialToken(
                        messageSource -> messageSource.createTokenSince(Duration.ofDays(31))
                );
        
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}
{% endtabs %}

**Token Claims**

As described [here](#tracking-tokens), a streaming processor should claim a token before it is allowed to perform any processing work.
There are several scenarios where a processor may keep the claim for too long.
This can occur when, for example, the event handling process is slow or encountered an exception.

In those scenarios, another processor can steal a token claim to proceed with processing.
There are a couple of configurable values that influence this process:

* `tokenClaimInterval` - Defines how long to wait between attempts to claim a segment.
  A processor uses this value to steal token claims from other processor threads. This value defaults to 5000 milliseconds.
* `eventAvailabilityTimeout` - Defines the time to "wait for events" before extending the claim.
  Only the Tracking Event Processor uses this. The value defaults to 1000 milliseconds.
* `claimExtensionThreshold` - Threshold to extend the claim in the absence of events.
  Only the Pooled Streaming Event Processor uses this. The value defaults 5000 milliseconds.

Consider the following snippets if you want to configure any of these values:

{% tabs %}
{% tab title="Tracking Processor - Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureTokenClaimValues(EventProcessingConfigurer processingConfigurer) {
        TrackingEventProcessorConfiguration tepConfig = 
                TrackingEventProcessorConfiguration.forSingleThreadedProcessing()
                                                   .andTokenClaimInterval(1000, TimeUnit.MILLISECONDS)
                                                   .andEventAvailabilityTimeout(2000, TimeUnit.MILLISECONDS);
        
        processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Tracking Processor - Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureTokenClaimValues(EventProcessingConfigurer processingConfigurer) {
          TrackingEventProcessorConfiguration tepConfig = 
                  TrackingEventProcessorConfiguration.forSingleThreadedProcessing()
                                                     .andTokenClaimInterval(1000, TimeUnit.MILLISECONDS)
                                                     .andEventAvailabilityTimeout(2000, TimeUnit.MILLISECONDS);
          
          processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureTokenClaimValues(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig = 
                (config, builder) -> builder.tokenClaimInterval(2000)
                                            .claimExtensionThreshold(3000);
        
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureTokenClaimValues(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig = 
                (config, builder) -> builder.tokenClaimInterval(2000)
                                            .claimExtensionThreshold(3000);
        
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}
{% endtabs %}

### Token Store

The `TokenStore` provides the CRUD operations for the `StreamingEventProcessor` to interact with `TrackingTokens`.
The streaming processor will use the store to construct, fetch and claim tokens.

When no token store is explicitly defined, an `InMemoryTokenStore` is used.
The `InMemoryTokenStore` is _not_ recommended in most production scenarios since it cannot maintain the progress through application shutdowns.
Unintentionally using the `InMemoryTokenStore` counts towards one of the unexpected scenarios where the framework creates an [initial token](#initial-tracking-token) on each application start-up.

The framework provides a couple of `TokenStore` implementations:

- `InMemoryTokenStore` - A `TokenStore` implementation that keeps the tokens in memory.
  This implementation does not suffice as a production-ready store in most applications.
- `JpaTokenStore` - A `TokenStore` implementation using JPA to store the tokens with.
  Expects that a table is constructed based on the `org.axonframework.eventhandling.tokenstore.jpa.TokenEntry`.
  It is easily auto-configurable with, for example, Spring Boot.
- `JdbcTokenStore` - A `TokenStore` implementation using JDBC to store the tokens with.
  Expects that the schema is constructed through the `JdbcTokenStore#createSchema(TokenTableFactory)` method.
  Several `TokenTableFactory` can be chosen here, like the `GenericTokenTableFactory`, `PostgresTokenTableFactory` or `Oracle11TokenTableFactory` implementation.
- `MongoTokenStore`- A `TokenStore` implementation using Mongo to store the tokens with.

> **Where to store Tokens?**
>
> Where possible, we recommend using a token store that stores tokens in the same database as to where the event handlers update the view models.
> This way, changes to the view model can be stored atomically with the changed tokens.
> Furthermore, it guarantees exactly-once processing semantics.

Note that you can configure the token store to use for a streaming processor in the `EventProcessingConfigurer`:

{% tabs %}
{% tab title="Axon Configuration API" %}
To configure a `TokenStore` for all processors:

```java
public class AxonConfig { 
    // ...
    public void registerTokenStore(EventProcessingConfigurer processingConfigurer) {
        TokenStore tokenStore = JpaTokenStore.builder()
                                             // …
                                             .build();
    
        processingConfigurer.registerTokenStore(config -> tokenStore);
    }
}
```

Alternatively, to configure a `TokenStore` for a specific processor, use:

```java
public class AxonConfig { 
    // ...
    public void registerTokenStore(EventProcessingConfigurer processingConfigurer, String processorName) {
        TokenStore tokenStore = JdbcTokenStore.builder()
                                              // …
                                              .build();
    
        processingConfigurer.registerTokenStore(processorName, config -> tokenStore);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
The default `TokenStore` implementation is defined based on dependencies available in Spring Boot, in the following order:

1. If any `TokenStore` bean is defined, that bean is used.
2. Otherwise, if an `EntityManager` is available, the `JpaTokenStore` is defined.
3. Otherwise, if a `DataSource` is defined, the `JdbcTokenStore` is created.
4. Lastly, the `InMemoryToken` store is used.

To override the TokenStore, either define a bean in a Spring `@Configuration` class:

```java
@Configuration
public class AxonConfig {
    // ...
    @Bean
    public TokenStore myTokenStore() {
        return JpaTokenStore.builder()
                            // …
                            .build();
    }
}
```

Alternatively, inject the `EventProcessingConfigurer`, which allows more fine-grained customization:

```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void registerTokenStore(EventProcessingConfigurer processingConfigurer) {
          TokenStore tokenStore = JdbcTokenStore.builder()
                                                // …
                                                .build();
          
          processingConfigurer.registerTokenStore(conf -> tokenStore)
                              // or, to define one for a specific processor:
                              .registerTokenStore("my-processor", conf -> tokenStore);
    }
}
```
{% endtab %}
{% endtabs %}

## Parallel Processing

Streaming processors can use [multiple threads](#thread-configuration) to process an event stream.
Using multiple threads allows the `StreamingEventProcessor` to more efficiently process batches of events.
As described [here](#tracking-tokens), a streaming processor's thread requires a claim on a tracking token to process events.

Thus, to be able to parallelize the load, we require several tokens per processor.
To that end, each token instance represents a _segment_ of the event stream, wherein each segment is identified through a number.
The stream segmentation approach ensures events aren't handled twice (or more), as that would otherwise introduce unintentional duplication.
Due to this, the Streaming Processor's API references segment claims instead of token claims throughout.

You can define the number of segments used by adjusting the `initialSegmentCount` property.
Only when a streaming processor starts for the first time can it initialize the number of segments to use.
This requirement follows from the fact each token represents a single segment.
Tokens, in turn, can only be initialized if they are not present yet, as is explained in more detail [here](#initial-tracking-token).

Whenever the number of segments should be adjusted during runtime, you can use the [split and merge](#splitting-and-merging-segments) functionality.
To adjust the number of initial segments, consider the following sample:

{% tabs %}
{% tab title="Tracking Processor - Axon Configuration API" %}
The default number of segments for a `TrackingEventProcessor` is one.

```java
public class AxonConfig {
    // ...
    public void configureSegmentCount(EventProcessingConfigurer processingConfigurer) {
        TrackingEventProcessorConfiguration tepConfig = 
                TrackingEventProcessorConfiguration.forParallelProcessing(2)
                                                   .andInitialSegmentsCount(2);
        
        processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Tracking Processor - Spring Boot AutoConfiguration" %}
The default number of segments for a `TrackingEventProcessor` is one.

```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureSegmentCount(EventProcessingConfigurer processingConfigurer) {
          TrackingEventProcessorConfiguration tepConfig = 
                  TrackingEventProcessorConfiguration.forParallelProcessing(2)
                                                     .andInitialSegmentsCount(2);
          
          processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Axon Configuration API" %}
The default number of segments for a `PooledStreamingEventProcessor` is sixteen.

```java
public class AxonConfig {
    // ...
    public void configureSegmentCount(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig =
                (config, builder) -> builder.initialSegmentCount(32);
        
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Spring Boot AutoConfiguration" %}
The default number of segments for a `PooledStreamingEventProcessor` is sixteen.

```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureSegmentCount(EventProcessingConfigurer processingConfigurer) {
        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig = 
                (config, builder) -> builder.initialSegmentCount(32);
        
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties File" %}
The default number of segments for a `TrackingEventProcessor` and `PooledStreamingEventProcessor` is one and sixteen, respectively.

```text
axon.eventhandling.processors.my-processor.mode=pooled
# Sets the initial number of segments
axon.eventhandling.processors.my-processor.initialSegmentCount=32
```
{% endtab %}
{% endtabs %}

> **Parallel Processing and Subscribing Event Processors**
>
> Note that [Subscribing Event Processor](subscribing.md) don't manage their own threads.
> Therefore, it is not possible to configure how they should receive their events.
> Effectively, they will always work on a sequential-per-aggregate basis, as that is generally the level of concurrency in the command handling component.

The Event Handling Components a processor is in charge of may have specific expectations on the event order.
The ordering is guaranteed when only a single thread is processing events.
Maintaining the ordering requires additional work when the stream is segmented for parallel processing, however.
When this is the case, the processor must ensure it sends the events to these handlers in that specific order.

Axon uses the `SequencingPolicy` for this.
The `SequencingPolicy` is a function that returns a value for any given message.
If the return value of the `SequencingPolicy` function is equal for two distinct event messages, it means that those messages must be processed sequentially.
By default, Axon components will use the `SequentialPerAggregatePolicy`, making it so that events published by the same aggregate instance will be handled sequentially.
Check out [this](#sequential-processing) section to understand how to influence the sequencing policy.

Each node running a streaming processor will attempt to start its configured amount of threads to start processing events.
The number of segments that a single thread can claim differ between the Tracking- and Pooled Streaming Event Processor.
A tracking processor can only claim a single segment per thread, whereas the pooled streaming processor can claim any amount of segments per thread.
These approaches provide different pros and cons for each implementation, which [this](#differences-between-tracking-and-pooled-streaming) section explains further.

### Sequential Processing

Even though events are processed asynchronously from their publisher, it is often desirable to process certain events in their publishing order.
In Axon, the `SequencingPolicy` controls this order.
The `SequencingPolicy` defines whether events must be handled sequentially, in parallel, or a combination of both.
Policies return a sequence identifier of a given event.

If the policy returns the _same_ identifier for two events, they must be handled sequentially by the Event Handling Component.
Thus, if the `SequencingPolicy` returns a _different_ value for two events, they may be processed concurrently.
Note that if the policy returns a `null` sequence identifier, the event may be processed in parallel with _any_ other events.

> ** Parallel Processing and Sagas**
>
> A [saga](../../sagas/README.md) instance is **never** invoked concurrently by multiple threads.
> Therefore, the `SequencingPolicy` is irrelevant for a saga.
> Axon will ensure each saga instance receives the events it needs to process in the order they have been published on the event bus.

Conceptually, the `SequencingPolicy` decides whether an event belongs to a given [segment](#parallel-processing).
Furthermore, Axon guarantees that Events that are part of the same segment are processed sequentially.

The framework provides several policies you can use out of the box:

* `SequentialPerAggregatePolicy` - The default policy.
  It will force domain events that were raised from the same aggregate to be handled sequentially.
  Thus, events from different aggregates may be handled concurrently.
  This policy is typically suitable for Event Handling Components that update details from aggregates in databases.
* `FullConcurrencyPolicy` - This policy will tell Axon that this Event Processor may handle all events concurrently.
  This means that there is no relationship between the events that require them to be processed in a particular order.
* `SequentialPolicy` - This policy tells Axon that it can process all events sequentially.
  Handling of an event will start when the handling of a previous event has finished.
* `PropertySequencingPolicy` - When configuring this policy, the user is required to provide a property name or property extractor function.
  This implementation provides a flexible solution to set up a custom sequencing policy based on a standard value present in your events.
  Note that this policy only reacts to properties present in the event class.
* `MetaDataSequencingPolicy` - When configuring this policy, the user is required to provide a `metaDataKey` to be used. This implementation provides a flexible solution to set up a custom sequencing policy based on a standard value present in your events' metadata.

Consider the following snippets when configuring a (custom) `SequencingPolicy`:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureSequencingPolicy(EventProcessingConfigurer processingConfigurer) {
          PropertySequencingPolicy<SomeEvent, String> mySequencingPolicy = 
                  PropertySequencingPolicy.builder(SomeEvent.class, String.class)
                                          .propertyName("myProperty")
                                          .build();
          
          processingConfigurer.registerDefaultSequencingPolicy(config -> mySequencingPolicy)
                              // or, to define one for a specific processor:
                              .registerSequencingPolicy("my-processor", config -> mySequencingPolicy);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureSequencingPolicy(EventProcessingConfigurer processingConfigurer,
                                          SequencingPolicy<EventMessage<?>> mySequencingPolicy) {

      processingConfigurer.registerDefaultSequencingPolicy(config -> mySequencingPolicy)
                          // or, to define one for a specific processor:
                          .registerSequencingPolicy("my-processor", config -> mySequencingPolicy);
    }

    @Bean
    public SequencingPolicy<EventMessage<?>> mySequencingPolicy() {
        return new SequentialPolicy();
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties File" %}
When we want to configure the `SequencingPolicy` in a properties file, we should provide a bean name:

```text
axon.eventhandling.processors.my-processor.mode=tracking
axon.eventhandling.processors.my-processor.sequencing-policy=mySequencingPolicy
```

This approach does require the bean name to be present in the Application Context of course:

```java
@Configuration
public class AxonConfig {
    // ...
    @Bean
    public SequencingPolicy<EventMessage<?>> mySequencingPolicy() {
        return new FullConcurrencyPolicy();
    }
}
```
{% endtab %}
{% endtabs %}

If the available policies do not suffice, you can define your own.
To that end, we should implement the `SequencingPolicy` interface.
This interface defines a single method, `getSequenceIdentifierFor(T)`, that returns the sequence identifier for a given event:

```java
public interface SequencingPolicy<T> {
    
    Object getSequenceIdentifierFor(T event);
}
```
### Thread Configuration

A Streaming Processor cannot process events in parallel without multiple threads configured.
We can process events in parallel by running [several nodes](#multi-node-processing) of an application. Or by configuring a `StreamingEventProcessor` to use several threads.
The following section describes the threading differences between the Tracking- and Pooled Streaming Event Processor.
These sections are followed up with samples on configuring multiple threads for the TEP and PSEP, respectively.

> **Thread and Segment Count**
> 
> Adjusting the number of threads will not automatically parallelize a Streaming Processor.
> A segment claim [is required](#parallel-processing) to let a thread process any events.
> Hence, increasing the thread count should be paired with adjusting the segment count.

#### Tracking Processor Threading

The `TrackingEventProcessor` uses a `ThreadFactory` to start the process of claiming segments.
It will use a single thread per segment it is able to claim until the processor exhausts the configured amount of threads.
Each thread will open a stream with the `StreamableMessageSource` and start processing events at their own speed.
Other segment operations, like [split and merge](#splitting-and-merging-segments), are processed by the thread owning the segment operated on.

Since the tracking processor can only claim a single segment per thread, segments may go unprocessed if there are more segments than threads.
Hence, we recommend setting the number of threads (on every node) higher than or equal to the total number of segments.

To increase event handling throughput, we recommend changing the number of threads.
How to do this is shown in the following sample:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureThreadCount(EventProcessingConfigurer processingConfigurer) {
        TrackingEventProcessorConfiguration tepConfig =
                TrackingEventProcessorConfiguration.forParallelProcessing(4)
                                                   .andInitialSegmentsCount(4);

        processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureThreadCount(EventProcessingConfigurer processingConfigurer) {
        TrackingEventProcessorConfiguration tepConfig =
                TrackingEventProcessorConfiguration.forParallelProcessing(4)
                                                   .andInitialSegmentsCount(4);

        processingConfigurer.registerTrackingEventProcessorConfiguration("my-processor", config -> tepConfig);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties File" %}
```text
axon.eventhandling.processors.my-processor.mode=tracking
axon.eventhandling.processors.my-processor.thread-count=4
axon.eventhandling.processors.my-processor.initial-segment-count=4
```
{% endtab %}
{% endtabs %}

#### Pooled Streaming Processor Threading

The `PooledStreamingEventProcessor` uses two threads pools instead of the single fixed set of threads used by the `TrackingEventProcessor`.
The first thread pool is in charge of opening a stream with the event source, claiming as _many_ segments as possible, and delegating all the work.

The work it coordinates is foremost the events to handle. Next to event coordination, it deals with segment operations like [split and merge](#splitting-and-merging-segments).
The component coordinating all the work is called the `Coordinator`.
This coordinator defaults to using a `ScheduledExecutorService` with a single thread, which suffices in most scenarios.

The second thread pool deals with all the segments the `Coordinator` of the pooled streaming processor could claim.
The `Coordinator` starts a `WorkPackage` for each segment and provides them the events to handle.
The work package will, in turn, invoke the Event Handling Components to process the events.
These packages run within the second thread pool, the so-called "worker executor" pool.
The worker-pool also defaults to `ScheduledExecutorService` with a single thread.

When you want to increase event handling throughput, we recommend changing the number of threads for the worker thread pool.
How to do this is shown in the following sample:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureThreadCount(EventProcessingConfigurer processingConfigurer) {
        // the "name" is the name of the processor, which can be used to define the thread factory name
        Function<String, ScheduledExecutorService> coordinatorExecutorBuilder =
                name -> Executors.newScheduledThreadPool(1, new AxonThreadFactory("Coordinator - " + name));

        Function<String, ScheduledExecutorService> workerExecutorBuilder =
                name -> Executors.newScheduledThreadPool(16, new AxonThreadFactory("Worker - " + name));

        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig =
                (config, builder) -> builder.coordinatorExecutor(coordinatorExecutorBuilder)
                                            .workerExecutorService(workerExecutorBuilder)
                                            .initialSegmentCount(32);

        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureThreadCount(EventProcessingConfigurer processingConfigurer) {
        // the "name" is the name of the processor, which can be used to define the thread factory name
        Function<String, ScheduledExecutorService> coordinatorExecutorBuilder = 
                name -> Executors.newScheduledThreadPool(1, new AxonThreadFactory("Coordinator - " + name));

        Function<String, ScheduledExecutorService> workerExecutorBuilder =
                name -> Executors.newScheduledThreadPool(16, new AxonThreadFactory("Worker - " + name));

        EventProcessingConfigurer.PooledStreamingProcessorConfiguration psepConfig =
                (config, builder) -> builder.coordinatorExecutor(coordinatorExecutorBuilder)
                                            .workerExecutorService(workerExecutorBuilder)
                                            .initialSegmentCount(32);
        
        processingConfigurer.registerPooledStreamingEventProcessorConfiguration("my-processor", psepConfig);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties File" %}
```text
axon.eventhandling.processors.my-processor.mode=pooled
# Only the thread count of the Worker can be influenced through a properties file!
axon.eventhandling.processors.my-processor.thread-count=16
axon.eventhandling.processors.my-processor.initial-segment-count=32
```
{% endtab %}
{% endtabs %}

#### Differences between Tracking and Pooled Streaming

Based on the threading approaches of the [tracking processor](#tracking-processor-threading) and [pooled streaming processor](#pooled-streaming-processor-threading), there are a couple of differences to note:

* **Open Event Streams** - The tracking processor will open a stream **per** segment it claims.
  The pooled streaming processor will always open a single event stream and delegate the events to the segment workers.
  Due to this, the tracking processor will use more I/O resources than the pooled streaming processor.
  However, the TEP's segments can move at their own speed as they open a dedicated event stream.
  The PSEP's segments will at least process as fast as the slowest segment in the set.

* **Segment Claims per Thread** - The tracking processor can only claim a single segment per thread.
  The pooled streaming processor can claim any amount of segments, regardless of the number of threads configured.
  The `maxClaimedSegments` is configurable if required (the defaults is `Short.MAX`).
  The fact the TEP can only claim a single segment per thread highlights a problem of that implementation.
  Events will go unprocessed if there are more segments than threads when using the tracking processor since events belong to a single segment.
  Furthermore, it makes dynamic scaling tougher since you cannot adjust the number of threads at runtime.
  Here we see significant benefits for using the PSEP instead of the TEP since it completely drops the "one segment per thread" policy.
  As such, partial processing is never a problem, the `PooledStreamingEventProcessor` would encounter.

* **Thread Pool Configuration** - The tracking processor does not allow sharing a thread pool between different instances.
  For the pooled streaming processor, a `ScheduledExecutorService` is configurable, which allows sharing the executor between different processor instances.
  Thus, the PSEP provides a higher level of flexibility towards optimizing the total amount of threads used within an application.
  The freedom in thread pool configuration is helpful when, for example, the number of different Event Processors in a single application increases.
  
> **Which Streaming Processor should I use?**
>
> In most scenarios, the `PooledStreamingEventProcessor` is the recommended processor implementation.
> We conclude this based on the segment-to-thread-count ratio, its ability to share thread pools, and the lower amount of opened event streams.
> 
> The `TrackingEventProcessor` will still be ideal if you anticipate the processing speed between segments to differ significantly.
> Also, if the application does not have too many processor instances, the need to share thread pools is loosened.

#### Multi-Node Processing

For streaming processors, it doesn't matter whether the threads handling the events are all running on the same node or on different nodes hosting the same \(logical\) processor.
When two (or more) instances of a streaming processor with the same name are active on different machines, they are considered two instances of the same logical processor.
Hence, it is not just a processor's own threads that compete for segments but also the processors on different application instances.

Thus in a multi-node setup, each processor instance will try to [claim segments](#parallel-processing), preventing events assigned to that segment from being processed on other nodes.
In this process, the processor updates the token by adding a node identifier when it claims a segment to enforce the claim. The node identifier is configurable on the `TokenStore`.
By default, it will use the JVM's name \(usually a combination of the hostname and process ID\) as the `nodeId`.

When in a multi-node scenario, often a fair distribution of the segments is desired.
Otherwise, the event processing load could be distributed unequally over the active instances.
There are roughly two approaches towards balancing the number of segments claimed per node:

1. Through the [Axon Server](../../../axon-server/introduction.md) Dashboard with the load balancing feature
2. Directly on a `StreamingEventProcessor`, with the `releaseSegment(int segmentId)` or `releaseSegment(int segmentId, long releaseDuration, TimeUnit unit)` method

When Axon Server is in place, we recommend using option one, as it is easiest to use.
Whenever Axon Server is not used, we can achieve load balancing by having a streaming processor release its segments.
Releasing segments is done by calling the `releaseSegment` method. When invoking `releaseSegment`, the `StreamingEventProcessor` will "let go of" the segment for some time.

As a consequence of releasing, another node will be able to claim the segment.
Due to this, releasing allows you to balance the load.
By default, the segment will be released for twice the [`tokenClaimInterval`](#token-configuration).

For those required to take the second approach, consider the following snippet as a form of guidance on how to release segments:

```java
class StreamingProcessorService {
    
    // The EventProcessingConfiguration allows access to all the configured EventProcessors
    private EventProcessingConfiguration processingConfiguration;

    // ...
    void releaseSegmentFor(String processorName, int segmentId) {
        // EventProcessingConfiguration#eventProcessor(String, Class) returns an optional of the event processor
        processingConfiguration.eventProcessor(processorName, StreamingEventProcessor.class)
                               .ifPresent(streamingProcessor -> streamingProcessor.releaseSegment(segmentId));
    }
}
```

### Splitting and Merging Segments

The Streaming Event Processor provides scalability by supporting [parallel processing](#parallel-processing).
Through this, it is possible to tune the processor's performance by [adjusting the number of threads](#thread-configuration).
However, only changing the number of threads is insufficient since the parallelization is dictated through the number of segments.

When there is a high event load, ideally, we increase the number of segments.
In turn, we can reduce the number of segments again if the load on the streaming processor decreases.
To change the number of segments at runtime, the _split and merge_ operations should be used.
Splitting and merging allow you to control the number of segments dynamically.

There are roughly two approaches to adjust the number of segments for a streaming processor:

1. Through the [Axon Server](../../../axon-server/introduction.md) Dashboard with the split and merge buttons
2. Directly on a `StreamingEventProcessor`, with the `splitSegment(int segmentId)` and `mergeSegment(int segmentId)` methods

When Axon Server is in place, we recommend using option one since it is easiest to use.
Whenever Axon Server is not used, and you want to adjust the number of segments, the split and merge methods should be accessible from within your application.
For those required to take the second approach, consider the following snippet as a form of guidance:

```java
class StreamingProcessorService {
    
    // The EventProcessingConfiguration allows access to all the configured EventProcessors
    private EventProcessingConfiguration processingConfiguration;

    // ...
    void splitSegmentFor(String processorName, int segmentId) {
        // EventProcessingConfiguration#eventProcessor(String, Class) returns an optional of the event processor
        processingConfiguration.eventProcessor(processorName, StreamingEventProcessor.class)
                               .ifPresent(streamingProcessor -> {
                                   // Use the result to check whether the operation succeeded
                                   CompletableFuture<Boolean> result =
                                           streamingProcessor.splitSegment(segmentId);
                               });
    }

    void mergeSegmentFor(String processorName, int segmentId) {
        processingConfiguration.eventProcessor(processorName, StreamingEventProcessor.class)
                               .ifPresent(streamingProcessor -> {
                                   // Use the result to check whether the operation succeeded
                                   CompletableFuture<Boolean> result =
                                           streamingProcessor.mergeSegment(segmentId);
                               });
    }
}
```

Note that if you are moving towards a solution using the `StreamingProcessorController`, there are a couple of points to consider.
When invoking the split/merge operation on a `StreamingEventProcessor`, that processor should be in charge of the segment you want to split or merge.
Thus, either the streaming processor already has a claim on the segment(s) or can claim the segment(s).
Without the claims, the processor will simply fail the split or merge operation.

It is advised to check which segments a streaming processor has a claim on. For that, [status of the processor](../../monitoring-and-metrics.md#event-tracker-status-a-idevent-tracker-statusa) is used.
The status information shows which segments a processor instance owns.
This guides which processor to invoke the split or merge on.

When doing a merge, the streaming processor should be in charge of **both** the provided `segmentId` and the segment the framework will merge it with.
We can calculate the segment identifier the provided `segmentId will be merged with through the`Segment#mergeableSegmentId` method.


> **Segment Selection Considerations**
>
> When splitting or merging through Axon Server, it chooses the most appropriate segment to split or merge for you.
> When using the Axon Framework API directly, the developer should deduce the segment to split or segments to merge by themselves:
> 
> * Split: for fair balancing, a split is ideally performed on the largest segment
> * Merge: for fair balancing, a merge is ideally performed on the smallest segment

## Replaying Events

A benefit of streaming events is that we can reopen the stream at any point in time.
Whenever some event handling components misbehaved, and the view models they update or actions they triggered should happen again, starting anew can be very useful.
Handling events again by adjusting the position on the stream is what's called "a replay," a feature supported by the `StreamingEventProcessor`.
The following sections describe how to [initiate a replay](#triggering-a-reset) and what [replay API](#replay-api) the framework provides.

### Triggering a Reset

The reset API revolves around the `resetTokens()` method and provides a couple of options:

* `resetTokens()` - 
  Simple reset, adjusting the `TrackingToken` to the configured [initial tracking token](#initial-tracking-token)
* `resetTokens(R resetContext)` - 
  Resets the `TrackingToken` to the configured [initial tracking token](#initial-tracking-token), providing the `resetContext` to the [`ResetHandlers`](#replay-api)  
* `resetTokens(Function<StreamableMessageSource<TrackedEventMessage<?>>, TrackingToken> initialTrackingTokenSupplier)` - 
  Resets the `TrackingToken` to the results of the `initialTrackingTokenSupplier`
* `resetTokens(Function<StreamableMessageSource<TrackedEventMessage<?>>, TrackingToken> initialTrackingTokenSupplier, R resetContext)` - 
  Resets the `TrackingToken` to the results of the `initialTrackingTokenSupplier`, providing the `resetContext` to the [`ResetHandlers`](#replay-api)
* `resetTokens(TrackingToken startPosition)` - 
  Resets the `TrackingToken` to the provided `startPosition`
* `resetTokens(TrackingToken startPosition, R resetContext)` - 
  Resets the `TrackingToken` to the provided `startPosition`, providing the `resetContext` to the [`ResetHandlers`](#replay-api)

> **Partial Replays**
>
> A replay does not always have to start "from the beginning of time."
> Partially replaying the event stream suffices for a lot of applications.
> 
> To perform a so-called "partial replay," you should provide the token at a specific point in time.
> The `StreamableMessageSource`'s [`createTokenAt(Instant)` and `createTokenSince(Duration)`](#initial-tracking-token) can be used for this.
> 
> If creating tokens based on time is not sufficient, but creating tokens based on the exact position is something that is more convenient, you could 
> create a `TrackingToken` providing the position and give it to `resetTokens(TrackingToken startPosition)` or 
> `resetTokens(TrackingToken startPosition, R resetContext)` methods. The concrete implementation of `TrackingToken` to provide depends on the `Token Store` being used.
> 
> Be mindful that when initiating a partial replay, the event handlers may handle an event in the middle of model construction.
> Hence, event handlers need to be "aware" that some events might not have been handled at all.
> Making the event handlers lenient (e.g., deal with missing data) or performing ad-hoc manual replays would help in that area.

As the method name suggests, the reset adjusts the [tracking token](#tracking-tokens) to a new position.
When starting a reset, the streaming processor is *required* to claim all its [segments](#parallel-processing).
All claims are required since the processor needs to update all tokens to their new position to start the replay.

To achieve this, the streaming event processor must be inactive when starting a reset.
Hence, it is required to be shut down first before invoking the `resetTokens` operation.
Once the reset was successful, the processor can be started up again.

Consider the following sample on how to trigger a reset within an application:

{% tabs %}
{% tab title="Reset without reset context" %}
```java
class StreamingProcessorController {
  
    private EventProcessingConfiguration processingConfiguration;
  
    // ...
    void resetTokensFor(String processorName) {
        // EventProcessingConfiguration#eventProcessor(String, Class) returns an optional of the event processor
        processingConfiguration.eventProcessor(processorName, StreamingEventProcessor.class)
                               .ifPresent(streamingProcessor -> {
                                   // shutdown this streaming processor
                                   streamingProcessor.shutDown();
                                   // reset the tokens to prepare the processor
                                   streamingProcessor.resetTokens();
                                   // start the processor to initiate the replay
                                   streamingProcessor.start();
                               });
    }
}
```
{% endtab %}

{% tab title="Reset with reset context" %}
```java
class StreamingProcessorController {
    
    private EventProcessingConfiguration processingConfiguration;

    // ...
    void resetTokensFor(String processorName, Object resetContext) {
        // EventProcessingConfiguration#eventProcessor(String, Class) returns an optional of the event processor
        processingConfiguration.eventProcessor(processorName, StreamingEventProcessor.class)
                               .ifPresent(streamingProcessor -> {
                                   // shutdown this streaming processor
                                   streamingProcessor.shutDown();
                                   // reset the tokens to prepare the processor
                                   streamingProcessor.resetTokens(resetContext);
                                   // start the processor to initiate the replay
                                   streamingProcessor.start(); 
                               });
  }
}
```
{% endtab %}
{% endtabs %}

> **Resets in multi-node environments**
>
> If you are in a [multi-node](#multi-node-processing) scenario, that means *all* nodes should shut down the `StreamingEventProcessor`.
> Otherwise, another node will pick up the segments released by the inactive processor instance.
> 
> Being able to shut down or start up all streaming processor instances is most easily achieved through the [Axon Server](../../../axon-server/introduction.md) Dashboard.
> The application's dashboard provides a "start" and "stop" button, which will start/stop the processor on every node.
> 
> When Axon Server is not used, you should construct a custom endpoint in your application.
> The `StreamingProcessorService` sample shared above would be ideal for adding a start and stop method. 

### Replay API

Initiating a replay through the `StreamingEventProcessor` opens up an API to tap into the process of replaying.
It is, for example, possible to define a `@ResetHandler`.
A processor will invoke `ResetHandler` annotated methods as a result of `StreamingEventProcessor#resetTokens`.
It provides a hook to prepare an Event Handling Component before the replay begins.

The following sample Event Handling Component shows the available replay API:

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

The `CardSummaryProjection` shows a couple of interesting things to take note of when it comes to "being aware" of a replay in progress:

1. An `@AllowReplay` can be used, situated either on an entire class or an `@EventHandler` annotated method.
   It defines whether the processor should invoke the given class or method when a replay is in transit.

2. In addition to allowing a replay, `@DisallowReplay` can also be used.
   Similar to `@AllowReplay`, you can place it on class level and methods.
   It serves to define whether a processor should **not** invoke the class or method when a replay is in transit.

3. To have more fine-grained control on what (not) to do during a replay, we can use the `ReplayStatus` parameter.
   The `ReplayStatus` is an additional parameter that we can add to `@EventHandler` annotated methods.
   It allows conditional operations in the event handlers based on whether a replay is taking place.

4. If it is necessary to perform certain pre-replay logic, such as clearing out a projection table, we can use the `@ResetHandler` annotation.
   It allows adding a "reset context" to provide more information on why the reset is taking place.
   To include a `resetContext` the `resetTokens(R resetContext)` method (or other methods containing the `resetContext` parameter) should be invoked.
   The type of the `resetContext` is up to the user.

## Multiple Event Sources

You can configure a Streaming Event Processor to use multiple sources to process events from.
When required to process events from several sources, we can configure a specific type of `StreamableMessageSource`: the `MultiStreamableMessageSource`.
The `MultiStreamableMessageSource` is useful when a streaming processor should act on the events from:
* several event stores,
* [multiple-contexts](../../../axon-server/administration/multi-context.md), or
* from different storage types (e.g., an Event Store and a Kafka Stream)

Having multiple sources means that there might be a choice of multiple events that the processor could consume at any given instant.
Therefore, you can specify a `Comparator` to choose between them.
The default implementation chooses the event with the oldest timestamp \(i.e., the event waiting for the longest\).

Using multiple sources also means that the streaming processor's polling interval needs to be divided between sources.
Some sources might use a strategy to optimize event discovery, thus minimizing overhead in establishing costly connections to the data sources.
To that end, you can choose which source the processor does most of the polling on using the `longPollingSource()` method in the builder.
This operation ensures one source consumes most of the polling interval while also checking intermittently for events on the other sources.
The `MultiStreamableMessageSource` defaults the `longPollingSource` to the last configured source.

Consider the following sample when constructing a `MultiStreamableMessageSource`:

```java
public class AxonConfig {
    // ...
    public MultiStreamableMessageSource buildMultiStreamableMessageSource(
            StreamableMessageSource<TrackedEventMessage<?>> eventSourceA,
            StreamableMessageSource<TrackedEventMessage<?>> eventSourceB,
            Comparator<Map.Entry<String, TrackedEventMessage<?>>> priorityA
    ) {
        return MultiStreamableMessageSource.builder()
                                           .addMessageSource("eventSourceA", eventSourceA)
                                           .addMessageSource("eventSourceB", eventSourceB)
                                           .longPollingSource("eventSourceA") // Overrides eventSourceB as the longPollingStream
                                           .trackedEventComparator(priorityA) // Where 'priorityA' is a comparator prioritizing events from eventSourceA
                                           .build();
    }
}
```

Assuming a `buildMultiStreamableMessageSource(...)` method is present, we can use the outcome to register a processor with the configuring `EventProcessingConfigurer`:

{% tabs %}
{% tab title="Tracking Processor - Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureTrackingProcessor(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer.registerTrackingEventProcessor(
                "my-processor", config -> buildMultiStreamableMessageSource(/*...*/)
        );
    }
}
```
{% endtab %}

{% tab title="Tracking Processor - Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configureTrackingProcessor(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer.registerTrackingEventProcessor(
                "my-processor", config -> buildMultiStreamableMessageSource(/*...*/)
        );
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configurePooledStreamingProcessor(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer.registerPooledStreamingEventProcessor(
                "my-processor", config -> buildMultiStreamableMessageSource(/*...*/)
        );
    }
}
```
{% endtab %}

{% tab title="Pooled Streaming Processor - Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // ...
    @Autowired
    public void configurePooledStreamingProcessor(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer.registerPooledStreamingEventProcessor(
                "my-processor", config -> buildMultiStreamableMessageSource(/*...*/)
        );
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties File" %}
When we want to configure the `StreamableMessageSource` in a properties file, we should provide a bean name:

```text
axon.eventhandling.processors.my-processor.mode=pooled
axon.eventhandling.processors.my-processor.source=multiStreamableMessageSource
```

This approach does require the bean name to be present in the Application Context of course:

```java
@Configuration
public class AxonConfig {
    // ...
    @Bean
    public MultiStreamableMessageSource multiStreamableMessageSource(
            StreamableMessageSource<TrackedEventMessage<?>> eventSourceA,
            StreamableMessageSource<TrackedEventMessage<?>> eventSourceB,
            Comparator<Map.Entry<String, TrackedEventMessage<?>>> priorityA
    ) {
        return MultiStreamableMessageSource.builder()
                                           .addMessageSource("eventSourceA", eventSourceA)
                                           .addMessageSource("eventSourceB", eventSourceB)
                                           .longPollingSource("eventSourceA")
                                           .trackedEventComparator(priorityA)
                                           .build();
    }
}
```
{% endtab %}
{% endtabs %}
