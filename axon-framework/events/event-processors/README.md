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
The Processing Group provides a level of configurable non-functional requirements, like [error handling](#processing-group---listener-invocation-error-handler) and the [sequencing policy](streaming.md#sequential-processing).

The Event Processors in turn is in charge of the Processing Group.
An Event Processor will control 1 to N Processing Groups, although in most cases there will be a one to one mapping.
Similar as the Event Handling Component, a Processing Group will belong to a single Event Processor.
This last layer allows definition of the type of Event Processor used, as well as concepts like the threading mode and a more fine-grained degree of [error handling](#event-processor---error-handler).

Event Processors come in roughly two forms: [Subscribing](subscribing.md) and [Streaming](streaming.md). 

Subscribing Event Processors subscribe themselves to a source of events and are invoked by the thread managed by the publishing mechanism. 
Streaming Event Processors, on the other hand, pull their messages from a source using a thread that it manages itself.

For more specifics on either type, their respective sections should be consulted.
The rest of this page dedicates itself towards describing the common concepts and configuration options of the Event Processor.

## Assigning handlers to processors

All processors have a name, which identifies a processor instance across JVM instances. 
Two processors with the same name are considered as two instances of the same processor.

All event handlers are attached to a processor whose name by default is the package name of the event handler's class.

For example, the following classes:

* `org.axonframework.example.eventhandling.MyHandler`,
* `org.axonframework.example.eventhandling.MyOtherHandler`, and
* `org.axonframework.example.eventhandling.module.ModuleHandler`

will trigger the creation of two processors:

1. `org.axonframework.example.eventhandling` with two handlers called `MyHandler` and `MyOtherHandler`
2. `org.axonframework.example.eventhandling.module` with the single handler `ModuleHandler`

Using the package name serves as a fair default, but using dedicated names for an Event Processor and/or the Processing Group is recommended.
The simplest approach to reach a clear naming scheme of your event handlers, is by using the `ProcessingGroup` annotation.
This annotation resembles the Processing Group level discussed in the [introduction](README.md).

The `ProcessingGroup` annotation requires insertion of a name and can only be set on the class.
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

If more control is required for grouping Event Handling Components, the [assignment rules](#event-handler-assignment-rules) should be consulted.

### Event Handler Assignment Rules

The Configuration API allows you to configure other strategies for assigning event handling classes to processors, or even assign specific handler instances to specific processors.
These assignment rules can be separated in roughly two groups: Event Handler to Processing Group and Processing Group to Event Processor.
Below is an exhaustive list of all the assignment rules which can be used through the `EventProcessingConfigurer`:

**Event Handler to Processing Group**

* `byDefaultAssignTo(String)` - defines the default Processing Group name an event handler will be assigned to.
  Will only be taken into account if there are no more specifics rules and if the `ProcessingGroup` annotation is not used.
* `byDefaultAssignHandlerInstancesTo(Function<Object, String>)` - defines a lambda invoked to assign an event handling instance to a desired Processing Group by returning that group's name.
  Will only be taken into account if there are no more specifics rules and if the `ProcessingGroup` annotation is not used.
* `byDefaultAssignHandlerTypesTo(Function<Class<?>, String>)` - defines a lambda invoked to assign an event handler type to a desired Processing Group by returning that group's name.
  Will only be taken into account if there are no more specifics rules and if the `ProcessingGroup` annotation is not used.
* `assignHandlerInstancesMatching(String, Predicate<Object>)` - assigns event handlers to the given Processing Group name based on a predicate ingesting an event handling instance.
  Uses a natural priority of zero. If an instance matches several criteria, the outcome is _undefined_.
* `assignHandlerTypesMatching(String, Predicate<Class<?>>)` - assigns event handlers to the given Processing Group name based on a predicate ingesting an event handler type.
  Uses a natural priority of zero. If an instance matches several criteria, the outcome is _undefined_. 
* `assignHandlerInstancesMatching(String, int, Predicate<Object>)` -  assigns event handlers to the given Processing Group name based on a predicate ingesting an event handling instance.
  Uses the given priority to decide on rule-ordering. The higher the priority, the more important the rule is.
  If an instance matches several criteria, the outcome is _undefined_.
* `assignHandlerTypesMatching(String, int, Predicate<Class<?>>)` - assigns event handlers to the given Processing Group name based on a predicate ingesting an event handler type.
  Uses the given priority to decide on rule-ordering. The higher the priority, the more important the rule is.
  If an instance matches several criteria, the outcome is _undefined_.

**Processing Group to Event Processor**

* `assignProcessingGroup(String, String)` - defines a given Processing Group name belongs to the given Event Processor's name.
* `assignProcessingGroup(Function<String, String>)` - defines a lambda invoked to assign a Processing Group name to a desired Event Processor by returning that processor's name.

### Ordering Event Handlers within a processor

To order event handlers within an Event Processor, the order in which event handlers are registered (as described in the [Registering Event Handlers](../event-handlers.md#registering-event-handlers) section) is guiding. 
Thus, the ordering in which event handlers will be called by an Event Processor for event handling is the same as their insertion ordering in the Configuration API.

If Spring is selected as the mechanism to wire everything, the ordering of the event handlers can be explicitly specified by adding the `@Order` annotation. 
This annotation should be placed at the class level of your event handler class, and an `integer` value should be provided to specify the ordering.

Note that it is **not possible** to order event handlers which are not a part of the same Event Processor.
Each Event Processor acts as its own component without any intervention from other Event Processors.

> **Ordering Considerations**
> 
> Although an order can be placed among event handlers within an Event Processor, it is recommended to go for separation instead.
> 
> Placing an overall ordering on event handlers means those components are inclined to interact with one another, introducing a form of coupling.
> Due to this the event handling process will become complex to manage (e.g. for new team members).
> Furthermore, embracing an ordering approach might lead to place _all_ event handlers in a global ordering, decreasing processing speeds in general.
> 
> In all, the ordering may definitely be used, but it is recommended to use it sparingly.

## Error Handling

Errors are inevitable in any application. 
Depending on where they happen, you may want to respond differently.

By default, exceptions raised by event handlers are caught in the [Processing Group layer](#processing-group---listener-invocation-error-handler), logged and processing continues with the next events. 
When an exception is thrown when a processor is trying to commit a transaction, update a [token](streaming.md#token-store), or in any other part of the process, the exception will be propagated. 

In case of a [Tracking Event Processor](streaming.md#tracking-event-processor), this means the processor will go into error mode, releasing any tokens and retrying at an incremental interval \(starting at 1 second, up to max 60 seconds\). 
A [Subscribing Event Processor](subscribing.md) will report a publication error to the component that provided the event.

To change this behavior there are two levels, the Processing Group and Event Processor respectively, at which you can customize how Axon deals with exceptions:

### Processing Group - Listener Invocation Error Handler

The component dealing with exceptions thrown from an event handling method, is called the `ListenerInvocationErrorHandler`.
By default, these exceptions are logged (with the `LoggingErrorHandler` implementation) and processing continues with the next handler or message.

The default `ListenerInvocationErrorHandler` used by each processing group can be customized.
Furthermore, the error handling behavior can also be configured per processing group:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
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
    // ...
    @Autowired
    public void configureProcessingGroupErrorHandling(EventProcessingConfigurer processingConfigurer) {
        // To configure a default ...
        processingConfigurer.registerDefaultListenerInvocationErrorHandler(conf -> /* create listener error handler */)
                            // ... or for a specific processing group: 
                            .registerListenerInvocationErrorHandler("my-processing-group", conf -> /* create listener error handler */);
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
When rethrown the exception will bubble up to the [Event Processor level](#event-processor---error-handler).

### Event Processor - Error Handler

Exceptions that occur outside the scope of an event handler, or have bubbled up from there, are handled by the `ErrorHandler`.
The default error handler is the `PropagatingErrorHandler`, which will rethrow any exceptions it catches.

How the Event Processor deals with a rethrown exception differs per implementation.
The behaviour for the Subscribing- and the Streaming Event Processor can respectively be found [here](subscribing.md#error-mode) and [here](streaming.md#error-mode).

A default `ErrorHandler` for all Event Processors or an `ErrorHandler` for a specific processors can be configured:

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
    // ...
    @Autowired
    public void configureProcessingGroupErrorHandling(EventProcessingConfigurer processingConfigurer) {
        // To configure a default ...
        processingConfigurer.registerDefaultErrorHandler(conf -> /* create error handler */)
                            // ... or for a specific processor: 
                            .registerErrorHandler("my-processor", conf -> /* create error handler */);
    }
}
```
{% endtab %}
{% endtabs %}

To provide a custom solution, the `ErrorHandler`'s single method needs to be implemented:

```java
public interface ErrorHandler {

    void handleError(ErrorContext errorContext) throws Exception;
}
```

Based on the provided `ErrorContext` object, you can decide to ignore the error, schedule retries, perform dead-letter-queue delivery or rethrow the exception.

## General processor configuration

Alongside [handler assignment](#assigning-handlers-to-processors) and [error handling](#error-handling), Event Processors allow configuration for other components too.
For [Subscribing](subscribing.md) and [Streaming](streaming.md) Event Processor specific options, their respective sections should be checked.
The remainder of this page will cover the generic configuration options for each Event Processor.

### Event Processor Builders

A lot of the configurable components for Event Processors can be accessed through the `EventProcessingConfigurer`.
Some times it is easier or preferable to provide an entire function to construct an Event Processor.
To that end, a custom `EventProcessorBuilder` can be configured:

```java
@FunctionalInterface
interface EventProcessorBuilder {

    // Note: the `EventHandlerInvoker` is the component which holds the event handling functions.
    EventProcessor build(String name, 
                         Configuration configuration, 
                         EventHandlerInvoker eventHandlerInvoker);
}
```

The `EventProcessorBuilder` functional interface provides the event processor's name, the `Configuration` and the `EventHandlerInvoker`, and requires an `EventProcessor` instance to be returned.
Note that any Axon component that need to be set on an Event Processor (e.g. an `EventStore`) can be retrieved from the `Configuration`.

The `EventProcessingConfigurer` provides two methods to configure an `EventProcessorBuilder`:

1. `registerEventProcessorFactory(EventProcessorBuilder)` - allows you to define a default factory method that creates event processors for which no explicit factories have been defined
2. `registerEventProcessor(String, EventProcessorBuilder)` - defines the factory method to use to create a processor with given `name`

### Event Handler Interceptors

Since the Event Processor is the invoker of event handling methods, it is a spot to configure [Message Handler Interceptors](../../messaging-concepts/message-intercepting.md) too.
As it is dedicated to event handling, the `MessageHandlerInterceptor` is required to deal with the `EventMessage`.
Differently put, an [EventHandlerInterceptor](../../messaging-concepts/message-intercepting.md#event-handler-interceptors) can be registered to Event Processors. 

The `EventProcessingConfigurer` provides two methods to configure `MessageHandlerInterceptor` instances:

- `registerDefaultHandlerInterceptor(BiFunction<Configuration, String, MessageHandlerInterceptor<? super EventMessage<?>>>)` - registers a default `MessageHandlerInterceptor` that will be configured on every Event Processor instance
- `registerHandlerInterceptor(String, Function<Configuration, MessageHandlerInterceptor<? super EventMessage<?>>>)` - registers a `MessageHandlerInterceptor` that will be configured for the Event Processor matching the given `String`

### Message Monitors

Any Event Processor instances provides the means to contain a Message Monitor. 
Message Monitors, as discussed in more detail [here](../../monitoring-and-metrics.md#metrics-a-idmetricsa), allow for monitoring the flow of messages throughout an Axon application.
For the Event Processor it will specifically deal with the events flowing through the Event Processor towards the event handling functions.

The `EventProcessingConfigurer` provides two approaches towards configuring a `MessageMonitor`:

- `registerMessageMonitor(String, Function<Configuration, MessageMonitor<Message<?>>>)` - registers the given `MessageMonitor` to the Event Processor matching the given `String`
- `registerMessageMonitorFactory(String, MessageMonitorFactory)` - registers the given `MessageMonitorFactory` to construct a `MessageMonitor` for the Event Processor matching the given `String`

The `MessageMonitorFactory` provides a more fine-grained approach, used through-out the Configuration API, to construct a `MessageMonitor`:

```java
@FunctionalInterface
public interface MessageMonitorFactory {
    
    MessageMonitor<Message<?>> create(Configuration configuration, 
                                      Class<?> componentType, 
                                      String componentName);
}
```

The `Configuration` may be used to retrieve required dependencies to construct the `MessageMonitor`.
The type and name reflect what infrastructure component the monitor is configured.
Whenever the `MessageMonitorFactory` is used to construct a `MessageMonitor` for an Event Processor, it is to be expected that the `componentType` is an `EventProcessor` implementation.
The `componentName` on the other hand would resemble the name of the Event Processor.

### Transaction Management

As components that deal with event handling, the Event Processor is a logical place to provide transaction configuration options.
Note that in the majority of the scenarios the defaults will suffice.
This section simply serves to show these options so that they may be adjusted if the application requires them to.

First of these configuration options is the `TransactionManager`.
Axon uses the `TransactionManager` to attach a transaction to every [Unit of Work](../../messaging-concepts/unit-of-work.md).
Within a Spring environment, the `TransactionManager` defaults to a `SpringTransactionManager`, which uses Spring's `PlatformTransactionManager` under the hood.
In non Spring environments, it would be wise to provide an implementation of this `TransactionManager` interface.
Such an implementation only requires the definition of the `TransactionManager#startTransaction()` method.
To adjust the transaction manager for an Event Processor, the `registerTransactionManager(String, Function<Configuration, TransactionManager>)` on the `EventProcessingConfigurer` should be used.

Secondly, the desired `RollbackConfiguration` can be adjusted per Event Processor.
It is the `RollbackConfiguration` that decide when a [Unit of Work](../../messaging-concepts/unit-of-work.md) should rollback the transaction.
The default `RollbackConfiguration` is to rollback on any type of `Throwable`; the [Unit of Work](../../messaging-concepts/unit-of-work.md) page describes the other options you can choose.
To adjust the default behaviour, the `registerRollbackConfiguration(String, Function<Configuration, RollbackConfiguration>)` function should be invoked on the `EventProcessingConfigurer`.
