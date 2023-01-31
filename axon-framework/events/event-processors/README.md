# Event Processors

[Event handlers](../event-handlers.md) define the business logic to be performed when an event is received.
_Event Processors_ are the components that take care of the technical aspects of that processing.
They start a [unit of work](../../messaging-concepts/unit-of-work.md) and possibly a transaction.
However, they also ensure that [correlation data](../../messaging-concepts/message-correlation.md) can be correctly attached to all messages created during event processing, among other non-functional requirements.

The image below depicts a representation of the organization of Event Processors and Event Handlers:

![Organization of Event Processors and Event Handlers](../../../.gitbook/assets/event-processors.png)

Axon has a layered approach towards organizing event handlers.
First, an event handler is positioned in a _Processing Group_.
Each event handler, or "Event Handling Component," will only ever belong to a single Processing Group.
The Processing Group provides a level of configurable non-functional requirements, like [error handling](#processing-group-listener-invocation-error-handler) and the [sequencing policy](streaming.md#sequential-processing).

The Event Processors, in turn, is in charge of the Processing Group.
An Event Processor will control 1 to N Processing Groups, although there will be a one-to-one mapping in most cases.
Similar to the Event Handling Component, a Processing Group will belong to a single Event Processor.
This last layer allows the definition of the type of Event Processor used and concepts like the threading model and a more fine-grained degree of [error handling](#event-processor-error-handler).

Event Processors come in roughly two forms: [Subscribing](subscribing.md) and [Streaming](streaming.md).

Subscribing Event Processors subscribe to a source of events and are invoked by the thread managed by the publishing mechanism.
Streaming Event Processors, on the other hand, pull their messages from a source using a thread that it manages itself.

For more specifics on either type, consult their respective sections [here](subscribing.md) and [here](streaming.md).
The rest of this page dedicates itself to describing the Event Processor's common concepts and configuration options.
Note that throughout, the `EventProcessingConfigurer` is used.
The `EventProcessingConfigurer` is part of Axon's Configuration API, dedicated to configuring Event Processors.

## Assigning handlers to processors

All processors have a name, which identifies a processor instance across JVM instances.
Two processors with the same name are considered as two instances of the same processor.

All event handlers are attached to a processor whose name by default is the package name of the event handler's class.
Furthermore, the default processor implementation used by Axon is the [Tracking Event Processor](streaming.md).
The (default) event processor used can be adjusted, as is shown in the [subscribing](subscribing.md#configuring) and [streaming](streaming.md#configuring) sections.

Event handlers, or Event Handling Components, come in roughly two flavors: "regular" \(singleton, stateless\) event handlers and [sagas](../../sagas/README.md).
[This](../event-handlers.md#registering-event-handlers) section describes the process to register an event handler, whereas [this](../../sagas/implementation.md#configuring-a-saga) page describes the saga registration process.

Now let us consider that the following event handlers have been registered:

* `org.axonframework.example.eventhandling.MyHandler`
* `org.axonframework.example.eventhandling.MyOtherHandler`
* `org.axonframework.example.eventhandling.module.ModuleHandler`

Without any intervention, this will trigger the creation of two processors, namely:

1. `org.axonframework.example.eventhandling` with two handlers called `MyHandler` and `MyOtherHandler`
2. `org.axonframework.example.eventhandling.module` with the single handler `ModuleHandler`

Using the package name serves as a suitable default, but using dedicated names for an Event Processor and/or the Processing Group is recommended.
The most straightforward approach to reaching a transparent naming scheme of your event handlers is by using the `ProcessingGroup` annotation.
This annotation resembles the Processing Group level discussed in the [introduction](README.md#event-processors).

The `ProcessingGroup` annotation requires the insertion of a name and can only be set on the class.
Let us adjust the previous sample by using this annotation instead of the package names for grouping handlers:

```java
@ProcessingGroup("my-handlers")
class MyHandler {
    // left out event handling functions...
}

@ProcessingGroup("my-handlers")
class MyOtherHandler{
    // ...
}

@ProcessingGroup("module-handlers")
class ModuleHandler {
    // ...
}
```

Using the `ProcessingGroup` annotation as depicted, we again construct two processors:

1. `my-handlers` with two handlers called `MyHandler` and `MyOtherHandler`
2. `module-handlers` with the single handler `ModuleHandler`

If more control is required to group Event Handling Components, we recommend consulting the [assignment rules](#event-handler-assignment-rules) section.

### Event Handler Assignment Rules

The Configuration API allows you to configure other strategies for assigning event handling classes to processors or assigning specific handler instances to particular processors.
We can separate these assignment rules into roughly two groups: Event Handler to Processing Group and Processing Group to Event Processor.
Below is an exhaustive list of all the assignment rules the `EventProcessingConfigurer` exposes:

**Event Handler to Processing Group**

* `byDefaultAssignTo(String)` - defines the default Processing Group name to assign an event handler to.
  It will only be taken into account if there are no more specifics rules and if the `ProcessingGroup` annotation is not present.
* `byDefaultAssignHandlerInstancesTo(Function<Object, String>)` - defines a lambda invoked to assign an event handling instance to a desired Processing Group by returning that group's name.
  It will only be taken into account if there are no more specifics rules and if the `ProcessingGroup` annotation is not present.
* `byDefaultAssignHandlerTypesTo(Function<Class<?>, String>)` - defines a lambda invoked to assign an event handler type to a desired Processing Group by returning that group's name.
  It will only be taken into account if there are no more specifics rules and if the `ProcessingGroup` annotation is not present.
* `assignHandlerInstancesMatching(String, Predicate<Object>)` - assigns event handlers to the given Processing Group name based on a predicate ingesting an event handling instance.
  The operation uses a natural priority of zero. If an instance matches several criteria, the outcome is _undefined_.
* `assignHandlerTypesMatching(String, Predicate<Class<?>>)` - assigns event handlers to the given Processing Group name based on a predicate ingesting an event handler type.
  The operation uses a natural priority of zero. If an instance matches several criteria, the outcome is _undefined_.
* `assignHandlerInstancesMatching(String, int, Predicate<Object>)` -  assigns event handlers to the given Processing Group name based on a predicate ingesting an event handling instance.
  Uses the given priority to decide on rule-ordering. The higher the priority value, the more important the rule is.
  If an instance matches several criteria, the outcome is _undefined_.
* `assignHandlerTypesMatching(String, int, Predicate<Class<?>>)` - assigns event handlers to the given Processing Group name based on a predicate ingesting an event handler type.
  Uses the given priority to decide on rule-ordering. The higher the priority, the more important the rule is.
  If an instance matches several criteria, the outcome is _undefined_.

**Processing Group to Event Processor**

* `assignProcessingGroup(String, String)` - defines a given Processing Group name that belongs to the given Event Processor's name.
* `assignProcessingGroup(Function<String, String>)` - defines a lambda invoked to assign a Processing Group name to the desired Event Processor by returning that processor's name.

### Ordering Event Handlers within a processor

To order event handlers within an Event Processor, the order in which event handlers are registered (as described in the [Registering Event Handlers](../event-handlers.md#registering-event-handlers) section) is guiding.
Thus, the ordering in which an Event Processor will call event handlers for event handling is the same as their insertion ordering in the Configuration API.

If we use Spring as the mechanism for wiring everything, we can explicitly specify the event handler component ordering by adding the `@Order` annotation.
This annotation is placed on the event handler class name, containing an `integer` value to specify the ordering.

Note that it is **not possible** to order event handlers belonging to different Event Processors.
Each Event Processor acts as an isolated component without any intervention from other Event Processors.

> **Ordering Considerations**
>
> Although we can place an order among event handlers within an Event Processor, separation of event handlers is recommended.
> 
> Placing an overall ordering on event handlers means those components are inclined to interact with one another, introducing a form of coupling.
> Due to this, the event handling process will become complex to manage (e.g., for new team members).
> Furthermore, embracing an ordering approach might lead to place _all_ event handlers in a global ordering, decreasing processing speeds in general.
> 
> In all, you are free to use an ordering, but we recommend using it sparingly.

## Error Handling

Errors are inevitable in any application.
Depending on where they happen, you may want to respond differently.

By default, exceptions raised by event handlers are caught in the [Processing Group layer](#processing-group-listener-invocation-error-handler), logged, and processing continues with the following events.
When an exception is thrown when a processor is trying to commit a transaction, update a [token](streaming.md#token-store), or in any other part of the process, the exception will be propagated.

In the case of a [Streaming Event Processor](streaming.md#error-mode), this means the processor will go into error mode, releasing any tokens and retrying at an incremental interval \(starting at 1 second, up to max 60 seconds\).
A [Subscribing Event Processor](subscribing.md#error-mode) will report a publication error to the component that provided the event.

To change this behavior, both the Processing Group and Event Processor level allow customization on how to deal with exceptions:

### Processing Group - Listener Invocation Error Handler

The component dealing with exceptions thrown from an event handling method is called the `ListenerInvocationErrorHandler`.
By default, these exceptions are logged (with the `LoggingErrorHandler` implementation), and processing continues with the next handler or message.

The default `ListenerInvocationErrorHandler` used by each processing group can be customized.
Furthermore, we can configure the error handling behavior per processing group:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // omitting other configuration methods...
    public void configureProcessingGroupErrorHandling(EventProcessingConfigurer processingConfigurer) {
        // To configure a default ...
        processingConfigurer.registerDefaultListenerInvocationErrorHandler(conf -> /* create listener error handler */)
                            // ... or for a specific processing group: 
                            .registerListenerInvocationErrorHandler("my-processing-group", conf -> /* create listener error handler */);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // omitting other configuration methods...
    @Bean
    public ConfigurerModule processingGroupErrorHandlingConfigurerModule() {
        return configurer -> configurer.eventProcessing(
                processingConfigurer -> processingConfigurer.registerDefaultListenerInvocationErrorHandler(
                                                                    conf -> /* create listener error handler */
                                                            )
                                                            // ... or for a specific processing group: 
                                                            .registerListenerInvocationErrorHandler(
                                                                    "my-processing-group",
                                                                    conf -> /* create listener error handler */
                                                            )
        );
    }
}
```
{% endtab %}
{% endtabs %}

It is easy to implement custom error handling behavior. 
The error handling method to implement provides the exception, the event that was handled, and a reference to the handler that was handling the message:

```java
public interface ListenerInvocationErrorHandler {

    void onError(Exception exception, 
                 EventMessage<?> event, 
                 EventMessageHandler eventHandler) throws Exception;
}
```

You can choose to retry, ignore or rethrow the exception.
The exception will bubble up to the [Event Processor level](#event-processor-error-handler) when rethrown.

### Event Processor - Error Handler

Exceptions occurring outside an event handler's scope, or have bubbled up from there, are handled by the `ErrorHandler`.
The default error handler is the `PropagatingErrorHandler`, which will rethrow any exceptions it catches.

How the Event Processor deals with a rethrown exception differ per implementation.
The behaviour for the Subscribing- and the Streaming Event Processor can respectively be found [here](subscribing.md#error-mode) and [here](streaming.md#error-mode).

We can configure a default `ErrorHandler` for all Event Processors or an `ErrorHandler` for specific processors:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void configureProcessingGroupErrorHandling(EventProcessingConfigurer processingConfigurer) {
        // To configure a default ...
        processingConfigurer.registerDefaultErrorHandler(conf -> /* create error handler */)
                            // ... or for a specific processor: 
                            .registerErrorHandler("my-processor", conf -> /* create error handler */);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // omitting other configuration methods...
    @Bean
    public ConfigurerModule processorErrorHandlingConfigurerModule() {
        return configurer -> configurer.eventProcessing(
                processingConfigurer -> processingConfigurer.registerDefaultErrorHandler(conf -> /* create error handler */)
                                                            // ... or for a specific processor: 
                                                            .registerErrorHandler(
                                                                    "my-processor",
                                                                    conf -> /* create error handler */
                                                            )
        );
    }
}
```
{% endtab %}
{% endtabs %}

For providing a custom solution, the `ErrorHandler`'s single method needs to be implemented:

```java
public interface ErrorHandler {

    void handleError(ErrorContext errorContext) throws Exception;
}
```

Based on the provided `ErrorContext` object, you can decide to ignore the error, schedule retries, perform dead-letter-queue delivery, or rethrow the exception.

### Dead-Letter Queue

Although configuring a [Listener Invocation Error Handler](#processing-group---listener-invocation-error-handler)
and [Error Handler](#event-processor---error-handler)
helps you to deal with exceptions when processing events, you still end up in an event handling stop.
When you only log the error and allow processing to proceed, you will most likely end up with missing data until you fix the predicament and [replay](streaming.md#replaying-events)
past events.
If you instead propagate the exception so the event processor keeps retrying, the event processor will stall entirely when the cause is consistent.

Although this behavior is sufficient on many occasions, sometimes it is beneficial if we can unblock event handling by parking the problematic event.
To that end, you can configure a dead-letter queue for a [processing group](#event-processors).

An essential concept of Axon Frameworks event processors is the maintenance of event ordering, even when you configure [parallel processing](streaming.md#parallel-processing).
A perfect example when this is a requirement is the need to handle events of the same aggregate in their publishing order.
Simply dead lettering one failed event would cause subsequent events in the sequence to be applied to inconsistent state.

It is thus of utmost importance that a dead-letter queue for events enqueues an event and any following events in the sequence.
To that end, the supported dead-letter queue is a so-called `SequencedDeadLetterQueue`.

Integral to its design is to allow for queueing failed events and events that belong to a faulty sequence.
It does so by maintaining a sequence identifier for each event, determined by the [sequencing policy](/axon-framework/events/event-processors/streaming.md#sequential-processing).

> **Is there support for Sagas?**
>
> Currently, there is *no* support for using a dead-letter queue for [sagas](/axon-framework/sagas/README.md).
> We've taken this decision as we cannot support a sequenced dead lettering approach as we do for regular event handling.
> 
> Furthermore, we cannot do this, as a saga's associations can vary widely between events.
> Due to this, the sequence of events may change, breaking this level of support.
> Hence, there's no way of knowing whether a next event in the stream does or does not belong to a saga.

Note that you *cannot* share a dead-letter queue between different processing groups.
Hence, each processing group you want to enable this behavior for should receive a unique dead-letter queue instance.

We currently provide the following dead-letter queue implementations:
* `InMemorySequencedDeadLetterQueue` - In-memory variant of the dead-letter queue. 
  Useful for testing purposes, but as it does not persist dead letters, it is unsuited for production environments.
* `JpaSequencedDeadLetterQueue` - JPA variant of the dead-letter queue. 
  It constructs a `dead_letter_entry` table where it persists failed-events in.
  The JPA dead-letter queue is a suitable option for production environments by persisting the dead letters.

#### Dead-Letter Queues and Idempotency

Before configuring a `SequencedDeadLetterQueue` it is vital to validate whether your event handling functions are idempotent.
As a processing group consists of several Event Handling Components (as explained in the intro of this chapter), some handlers may succeed in event handling while others will not.
As a configured dead-letter queue does not stall event handling, a failure in one Event Handling Component does not cause a rollback for other event handlers.
Furthermore, as the dead-letter support is on the processing group level, [dead-letter processing](#processing-dead-letter-sequences) will invoke *all* event handlers for that event within the processing group.

Thus, if your event handlers are not idempotent, processing letters may result in undesired side effects.
Hence, we strongly recommend making your event handlers idempotent when using the dead-letter queue.
The principle of exactly-once delivery is no longer guaranteed; at-least-once delivery is the reality to cope with.

#### Configuring a sequenced Dead-Letter Queue

A `JpaSequencedDeadLetterQueue` configuration example:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // omitting other configuration methods...
    public void configureDeadLetterQueue(EventProcessingConfigurer processingConfigurer) {
        // Replace "my-processing-group" for the processing group you want to configure the DLQ on. 
        processingConfigurer.registerDeadLetterQueue(
                "my-processing-group",
                config -> JpaSequencedDeadLetterQueue.builder()
                                                     .processingGroup("my-processing-group")
                                                     .maxSequences(256)
                                                     .maxSequenceSize(256)
                                                     .entityManagerProvider(config.getComponent(EntityManagerProvider.class))
                                                     .transactionManager(config.getComponent(TransactionManager.class))
                                                     .serializer(config.serializer())
                                                     .build()
        );
    }
}
```
{% endtab %}
{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // omitting other configuration methods...
    @Bean
    public ConfigurerModule deadLetterQueueConfigurerModule() {
        // Replace "my-processing-group" for the processing group you want to configure the DLQ on.
        return configurer -> configurer.eventProcessing().registerDeadLetterQueue(
                "my-processing-group",
                config -> JpaSequencedDeadLetterQueue.builder()
                                                     .processingGroup("my-processing-group")
                                                     .maxSequences(256)
                                                     .maxSequenceSize(256)
                                                     .entityManagerProvider(config.getComponent(EntityManagerProvider.class))
                                                     .transactionManager(config.getComponent(TransactionManager.class))
                                                     .serializer(config.serializer())
                                                     .build()
        );
    }
}
```
{% endtab %}
{% endtabs %}

You can set the maximum number of saved sequences (defaults to 1024) and the maximum number of dead letters in a sequence (also defaults to 1024).
If either of these thresholds is exceeded, the queue will throw a `DeadLetterQueueOverflowException`.
This exception means the processing group will stop processing new events altogether.
Thus, the processing group moves back to the behavior described at the start of the [Error Handling](#error-handling) section.

#### Processing Dead-Letter Sequences

Once you resolve the problem that led to dead lettering events, we can start processing the dead letters.
We recommend using the `SequencedDeadLetterProcessor` interface for this, as it processes an entire dead-letter _sequence_ instead of single dead-letter entries.
It will thus ensure the event order is maintained during the retry.

The `SequencedDeadLetterProcessor` provides two operations to process dead letters:

1. `boolean processAny()` - Process the oldest dead-letter sequence.
   Returns `true` if it processes a sequence successfully.
2. `boolean process(Predicate<DeadLetter<? extends EventMessage<?>>)` - Process the oldest dead-letter sequence matching the predicate.
   Note that the predicate only filters based on a sequence's *first* entry.
   Returns `true` if it processes a sequence successfully.

If the processing of a dead letter fails, the event will be offered to the dead-letter queue again.
How the dead-lettering process reacts to this depends on the [enqueue policy](#dead-letter-enqueue-policy).

You can retrieve a `SequencedDeadLetterProcessor` from the `EventProcessingConfiguration` based on a processing group name *if* you have configured a dead-letter queue for this processing group.
Below are a couple of examples of how to process dead-letter sequences:

{% tabs %}
{% tab title="Process the oldest dead-letter sequence matching `ErrorEvent`" %}
```java
public class DeadletterProcessor {
    
    private EventProcessingConfiguration config;
    
    public void retryErrorEventSequence(String processingGroup) {
        config.sequencedDeadLetterProcessor(processingGroup)
              .ifPresent(letterProcessor -> letterProcessor.process(
                      deadLetter -> deadLetter.message().getPayload() instanceof ErrorEvent
              ));
    }
}
```
{% endtab %}
{% tab title="Process the oldest dead-letter sequence in the queue" %}
```java
public class DeadletterProcessor {
    
    private EventProcessingConfiguration config;
    
    public void retryAnySequence(String processingGroup) {
        config.sequencedDeadLetterProcessor(processingGroup)
              .ifPresent(SequencedDeadLetterProcessor::processAny);
    }
}
```
{% endtab %}
{% tab title="Process all dead-letter sequences in the queue" %}
```java
public class DeadletterProcessor {
    
    private EventProcessingConfiguration config;
    
    public void retryAllSequences(String processingGroup) {
        Optional<SequencedDeadLetterProcessor<EventMessage<?>>> optionalLetterProcessor = 
                config.sequencedDeadLetterProcessor(processingGroup);
        if (!optionalLetterProcessor.isPresent()) {
            return;
        }
        SequencedDeadLetterProcessor<EventMessage<?>> letterProcessor = optionalLetterProcessor.get();
        
        // Retrieve all the dead lettered event sequences:
       Iterable<Iterable<DeadLetter<? extends EventMessage<?>>>> deadLetterSequences = 
               config.deadLetterQueue(processingGroup)
                     .map(SequencedDeadLetterQueue::deadLetters)
                     .orElseThrow(() -> new IllegalArgumentException("No such Processing Group"));
       
       // Iterate over all sequences:
       for (Iterable<DeadLetter<? extends EventMessage<?>>> sequence : deadLetterSequences) {
           Iterator<DeadLetter<? extends EventMessage<?>>> sequenceIterator = sequence.iterator();
           String firstLetterId = sequenceIterator.next()
                                                  .message()
                                                  .getIdentifier();
           
           // SequencedDeadLetterProcessor#process automatically retries an entire sequence.
           // Hence, we only need to filter on the first entry of the sequence:
          letterProcessor.process(deadLetter -> deadLetter.message().getIdentifier().equals(firstLetterId));
       }
    }
}
```
{% endtab %}
{% endtabs %}

#### Dead-Letter attributes

A dead letter contains the following attributes:

| attribute        | type                | description                                                                                                                            |
|------------------|---------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `message`        | `EventMessage`      | The `EventMessage` for which handling failed. The message contains your event, among other `Message` properties.                       |
| `cause`          | `Optional<Cause>`   | The cause for the message to be dead lettered. Empty if the letter is enqueued because it is part of a sequence.                       |
| `enqueuedAt`     | `Instant`           | The moment in time when the event was enqueued in a dead-letter queue.                                                                 |
| `lastTouched`    | `Instant`           | The moment in time when this letter was last touched. Will equal the `enqueuedAt` value if this letter is enqueued for the first time. |
| `diagnostics`    | `MetaData`          | The diagnostic `MetaData` concerning this letter. Filled through the [enqueue policy](#dead-letter-enqueue-policy).                    |

#### Dead-Letter Enqueue Policy

By default, when you configure a dead-letter queue and event handling fails, the event is dead-lettered.
However, you might not want all event failures to result in dead-lettered entries.
Similarly, when [letter processing](#processing-dead-letter-sequences) fails, you might want to reconsider whether you want to enqueue the letter again.

To that end, you can configure a so-called `EnqueuePolicy`.
The enqueue policy ingests a `DeadLetter` and a cause (`Throwable`) and returns an `EnqueueDecision`.
The `EnqueueDecision`, in turn, describes if the framework should or should not enqueue the dead letter.

You can customize the dead-letter policy to exclude some events when handling fails.
As a consequence, these events will be skipped.
Note that Axon Framework invokes the policy on initial event handling *and* on [dead-letter processing](#processing-dead-letter-sequences).

Reevaluating the policy after processing failed may be essential to ensure a dead letter isn't stuck in the queue forever.
To deal with this scenario, you can attach additional diagnostic information to the dead letter through the policy. 
For example to add a number of retries to the dead letter to base your decision on.
See the sample `EnqueuePolicy` below for this:

```java
public class CustomEnqueuePolicy implements EnqueuePolicy<EventMessage<?>> {

    @Override
    public EnqueueDecision<EventMessage<?>> decide(DeadLetter<? extends EventMessage<?>> letter, Throwable cause) {
        if (cause instanceof NullPointerException) {
            // It's pointless:
            return Decisions.doNotEnqueue();
        }

        final int retries = (int) letter.diagnostics().getOrDefault("retries", -1);
        if (letter.message().getPayload() instanceof ErrorEvent) {
            // Important and new entry:
            return Decisions.enqueue(cause);
        }
        if (retries < 10) {
            // Let's continue and increase retries:
            return Decisions.requeue(cause, l -> l.diagnostics().and("retries", retries + 1));
        }

        // Exhausted all retries:
        return Decisions.evict();
    }
}
```

The `Decisions` utility class provides the most reasonable decisions, but you are free to construct your own `EnqueueDecision` when necessary.
See the following example for configuring our custom policy: 

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // omitting other configuration methods...
    public void configureEnqueuePolicy(EventProcessingConfigurer configurer) {
        // Replace "my-processing-group" for the processing group you want to configure the policy on.
        configurer.registerDeadLetterPolicy("my-processing-group", config -> new MyEnqueuePolicy());
    }
}
```
{% endtab %}
{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
    // omitting other configuration methods...
    @Bean
    public ConfigurerModule enqueuePolicyConfigurerModule() {
        // Replace "my-processing-group" for the processing group you want to configure the policy on.
        return configurer -> configurer.eventProcessing()
                                       .registerDeadLetterPolicy("my-processing-group", config -> new MyEnqueuePolicy());
    }
}
```
{% endtab %}
{% endtabs %}

## General processor configuration

Alongside [handler assignment](#assigning-handlers-to-processors) and [error handling](#error-handling), Event Processors allow configuration for other components too.
For [Subscribing](subscribing.md#configuring) and [Streaming](streaming.md#configuring) Event Processor specific options, their respective sections should be checked.
The remainder of this page will cover the generic configuration options for each Event Processor.

### Event Processor Builders

The `EventProcessingConfigurer` provides access to a lot of configurable components for Event Processors.
Sometimes it is easier or preferable to provide an entire function to construct an Event Processor, however.
To that end, we can configure a custom `EventProcessorBuilder`:

```java
@FunctionalInterface
interface EventProcessorBuilder {

    // Note: the `EventHandlerInvoker` is the component which holds the event handling functions.
    EventProcessor build(String name, 
                         Configuration configuration, 
                         EventHandlerInvoker eventHandlerInvoker);
}
```

The `EventProcessorBuilder` functional interface provides the event processor's name, the `Configuration` and the `EventHandlerInvoker`, and requires returning an `EventProcessor` instance.
Note that any Axon component that an Event Processor requires (e.g., an `EventStore`) is retrievable from the `Configuration`.

The `EventProcessingConfigurer` provides two methods to configure an `EventProcessorBuilder`:

1. `registerEventProcessorFactory(EventProcessorBuilder)` - allows you to define a default factory method that creates event processors for which no explicit factories are defined
2. `registerEventProcessor(String, EventProcessorBuilder)` - defines the factory method to use to create a processor with given `name`

### Event Handler Interceptors

Since the Event Processor is the invoker of event handling methods, it is a spot to configure [Message Handler Interceptors](../../messaging-concepts/message-intercepting.md) too.
Since Event Processors are dedicated to event handling, the `MessageHandlerInterceptor` is required to deal with an `EventMessage`.
Differently put, an [EventHandlerInterceptor](../../messaging-concepts/message-intercepting.md#event-handler-interceptors) can be registered to Event Processors.

The `EventProcessingConfigurer` provides two methods to configure `MessageHandlerInterceptor` instances:

- `registerDefaultHandlerInterceptor(BiFunction<Configuration, String, MessageHandlerInterceptor<? super EventMessage<?>>>)` - registers a default `MessageHandlerInterceptor` that will be configured on every Event Processor instance
- `registerHandlerInterceptor(String, Function<Configuration, MessageHandlerInterceptor<? super EventMessage<?>>>)` - registers a `MessageHandlerInterceptor` that will be configured for the Event Processor matching the given `String`

### Message Monitors

Any Event Processor instance provides the means to contain a Message Monitor.
Message Monitors (discussed in more detail [here](../../monitoring/metrics.md)) allow for monitoring the flow of messages throughout an Axon application.
For Event Processors, the message monitor deals explicitly with the events flowing through the Event Processor towards the event handling functions.

The `EventProcessingConfigurer` provides two approaches towards configuring a `MessageMonitor`:

- `registerMessageMonitor(String, Function<Configuration, MessageMonitor<Message<?>>>)` - registers the given `MessageMonitor` to the Event Processor matching the given `String`
- `registerMessageMonitorFactory(String, MessageMonitorFactory)` - registers the given `MessageMonitorFactory` to construct a `MessageMonitor` for the Event Processor matching the given `String`

The `MessageMonitorFactory` provides a more fine-grained approach, used throughout the Configuration API, to construct a `MessageMonitor`:

```java
@FunctionalInterface
public interface MessageMonitorFactory {
    
    MessageMonitor<Message<?>> create(Configuration configuration, 
                                      Class<?> componentType, 
                                      String componentName);
}
```

We can use the `Configuration` to retrieve the required dependencies to construct the `MessageMonitor`.
The type and name reflect which infrastructure component the factory constructs a monitor for.
Whenever you use the `MessageMonitorFactory` to construct a `MessageMonitor` for an Event Processor, the factory expects the `componentType` to be an `EventProcessor` implementation.
The `componentName`, on the other hand, would resemble the name of the Event Processor.

### Transaction Management

As components that deal with event handling, the Event Processor is a logical place to provide transaction configuration options.
Note that in the majority of the scenarios, the defaults will suffice.
This section simply serves to show these options to allow adjustment if the application requires it.

The first of these is the `TransactionManager`.
Axon uses the `TransactionManager` to attach a transaction to every [Unit of Work](../../messaging-concepts/unit-of-work.md).
Within a Spring environment, the `TransactionManager` defaults to a `SpringTransactionManager`, which uses Spring's `PlatformTransactionManager` under the hood.
In non Spring environments, it would be wise to build a `TransactionManager` implement if transaction management is required, of course.
Such an implementation only requires the definition of the `TransactionManager#startTransaction()` method.
To adjust the transaction manager for an Event Processor, the `registerTransactionManager(String, Function<Configuration, TransactionManager>)` on the `EventProcessingConfigurer` should be used.

Secondly, you can adjust the desired `RollbackConfiguration` per Event Processor.
It is the `RollbackConfiguration` that decide when a [Unit of Work](../../messaging-concepts/unit-of-work.md) should rollback the transaction.
The default `RollbackConfiguration` is to rollback on any type of `Throwable`; the [Unit of Work](../../messaging-concepts/unit-of-work.md) page describes the other options you can choose.
To adjust the default behaviour, the `registerRollbackConfiguration(String, Function<Configuration, RollbackConfiguration>)` function should be invoked on the `EventProcessingConfigurer`.

