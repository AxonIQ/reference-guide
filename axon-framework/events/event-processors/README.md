# Event Processors

[Event handlers](../event-handlers.md) define the business logic to be performed when an event is received. 
_Event Processors_ are the components that take care of the technical aspects of that processing. 
They start a [unit of work](../../messaging-concepts/unit-of-work.md) and possibly a transaction. 
However, they also ensure that [correlation data](../../monitoring-and-metrics.md#correlation-data-a-idcorrelation-dataa) can be correctly attached to all messages created during event processing, among other non-functional requirements.

A representation of the organization of Event Processors and Event Handlers is depicted below:

![Organization of Event Processors and Event Handlers](../../../.gitbook/assets/event-processors.png)

Axon has a layered approach towards organizing event handlers.
First, an event handler will be positioned in a _Processing Group_.
Each event handler, or "Event Handling Component", will only ever belong to a single Processing Group. 
The Processing Group provides a level of configurable non-functional requirements, like [error handling](#exceptions-raised-by-event-handler-methods) and the [sequencing policy](streaming.md#sequential-processing).

The Event Processors in turn is in charge of the Processing Group.
An Event Processor will control 1 to N Processing Groups, although in most cases there will be a one to one mapping.
Similar as the Event Handling Component, a Processing Group will belong to a single Event Processor.
This last layer allows definition of the type of Event Processor used, as well as concepts like the threading mode and a more fine-grained degree of [error handling](#exceptions-during-processing).

Event Processors come in roughly two forms: [Subscribing](subscribing.md) and [Streaming](streaming.md). 

Subscribing Event Processors subscribe themselves to a source of events and are invoked by the thread managed by the publishing mechanism. 
Streaming Event Processors, on the other hand, pull their messages from a source using a thread that it manages itself.

For more specifics on either type, their respective sections should be consulted.
The rest of this page dedicates itself towards describing the common concepts and configuration options of the Event Processor.

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

To order Event Handlers within an Event Processor, the order in which Event Handlers are registered \(as described in the [Registering Event Handlers](../event-handlers.md#registering-event-handlers) section\) is guiding. Thus, the ordering in which Event Handlers will be called by an Event Processor for Event Handling is the same as their insertion ordering in the configuration API.

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

> **A new Saga's Event Stream position**
>
> The Saga `TrackingEventProcessor` will by default start its token at the head of the stream.

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

## Distributing Events

Distributing events in inter-process environments is supported with Axon Server by default.

Alternatively, you can choose other components that you can find in one of the extension modules \(Spring AMQP, Kafka\).
