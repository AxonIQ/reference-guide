# Subscribing Event Processor

The `SubscribingEventProcessor`, or Subscribing Processor for short, is a type of [Event Processor](README.md).
As any Event Processor, it serves as the technical aspect to handle events by invoking the event handlers written in an Axon application.

The Subscribing Processor defines itself by receiving the events from a `SubscribableMessageSource`.
The `SubscribableMessageSource` is an infrastructure component to register a Subscribing Processor too.

After registration to the `SubscribableMessageSource`, the message source gives the events to the `SubscribingEventProcessor` in the order they are received.
Examples of a `SubscribableMessageSource` are the `EventBus` or the [AMQP Extension](../../../extensions/spring-amqp.md).
Both the `EventBus` and AMQP Extension are simple message bus solutions for events.

The simple bus solution makes the `SubscribableMessageSource` and thus the Subscribing Processor an approach to only receive _current_ events.
Operations like [replaying](streaming.md#replaying-events) are, therefore, not an option for any Subscribing Processor as long as the `SubscribableMessageSource` follows this paradigm.

Furthermore, the message source will use the same thread that receives the events to invoke the registered Subscribing Processors.
When the `EventBus` is, for example, used as the message source, this means that the event publishing thread is the same one handling the event in the Subscribing Processor.

Although this approach deserves a spot within the framework, most scenarios require further decoupling of components by separating the threads as well.
When, for example, an application requires event processing parallelization to get a higher performance, this can be a blocker.
This predicament is why the `SubscribingEventProcessor` is not the default in Axon Framework.

Instead, the "Tracking Event Processor" (a [Streaming Processor](streaming.md#streaming-event-processor) implementation) takes up that role.
It provides greater flexibility for developers for configuring the event processor in greater detail.

> **Subscribing Processor Use Cases**
>
> Although the `SubscribingEventProcessor` does not support easy parallelization or replays, there are still scenarios when it is beneficial.
>
> When a model, for example, should be updated within the same thread that published the event, the Subscribing Processor becomes a reasonable solution.
> In combination with Axon's [AMQP](../../../extensions/spring-amqp.md) or [Kafka](../../../extensions/kafka.md) extension, some of these concerns are alleviated too, making it a viable option.

## Configuring

Other than configuring that an app uses a Subscribing Event Processor, everything is covered [here](README.md#general-processor-configuration).
Firstly, to specify that a new Event Processors should default to a `SubscribingEventProcessor`, you can use the `usingSubscribingEventProcessors` method:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureProcessorDefault(EventProcessingConfigurer processingConfigurer) {
        processingConfigurer.usingSubscribingEventProcessors();
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
        processingConfigurer.usingSubscribingEventProcessors();
    }
}
```
{% endtab %}
{% endtabs %}

For a specific Event Processor to be a Subscribing instance, `registerSubscribingEventProcessor` is used:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureSubscribingProcessors(EventProcessingConfigurer processingConfigurer) {
        // To configure a processor to be subscribing ...
        processingConfigurer.registerSubscribingEventProcessor("my-processor")
                            // ... to define a specific SubscribableMessageSource ... 
                            .registerSubscribingEventProcessor("my-processor", conf -> /* create/return SubscribableMessageSource */);
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
    public void configureSubscribingProcessors(EventProcessingConfigurer processingConfigurer) {
        // To configure a processor to be subscribing ...
        processingConfigurer.registerSubscribingEventProcessor("my-processor")
                            // ... to define a specific SubscribableMessageSource ... 
                            .registerSubscribingEventProcessor("my-processor", conf -> /* create/return SubscribableMessageSource */);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties file" %}
A properties file allows the configuration of some fields on an Event Processor.
Do note that the Java configuration provides more degrees of freedom.

```text
axon.eventhandling.processors.my-processor.mode=subscribing
axon.eventhandling.processors.my-processor.source=eventBus
```

If the name of an event processor contains periods `.`, use the map notation:

```text
axon.eventhandling.processors[my.processor].mode=subscribing
axon.eventhandling.processors[my.processor].source=eventBus
```
{% endtab %}
{% endtabs %}

## Error Mode

Whenever the [error handler](README.md#event-processor-error-handler) rethrows an exception, the `SubscribingEventProcessor` will have it bubble up to the publishing component of the event.
Providing the exception to the event publisher allows the publishing component to deal with it accordingly.
