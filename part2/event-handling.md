Event Handling
===============

Event listeners are the components that act on incoming events. They typically execute logic based on decisions that have been made by the command model. Usually, this involves updating view models or forwarding updates to other components, such as 3rd party integrations. In some cases Event Handlers will throw Events themselves based on (patterns of) Events that it received, or even send Commands to trigger further changes. 

Defining Event Handlers
-----------------------

In Axon, an object may declare a number of Event Handler methods, by annotating them with `@EventHandler`. The declared parameters of the method define which events it will receive.

Axon provides out-of-the-box support for the following parameter types:

* The first parameter is always the payload of the Event Message. In the case the Event Handlers doesn't need access to the payload of the message, you can specify the expected payload type on the `@EventHandler` annotation. When specified, the first parameter is resolved using the rules specified below. Do not configure the payload type on the annotation if you want the payload to be passed as a parameter.

* Parameters annotated with `@MetaDataValue` will resolve to the Meta Data value with the key as indicated on the annotation. If `required` is `false` (default), `null` is passed when the meta data value is not present. If `required` is `true`, the resolver will not match and prevent the method from being invoked when the meta data value is not present.

* Parameters of type `MetaData` will have the entire `MetaData` of an `EventMessage` injected.

* Parameters annotated with `@Timestamp` and of type `java.time.Instant` (or `java.time.temporal.Temporal`) will resolve to the timestamp of the `EventMessage`. This is the time at which the Event was generated.

* Parameters annotated with `@SequenceNumber` and of type `java.lang.Long` or `long` will resolve to the `sequenceNumber` of a `DomainEventMessage`. This provides the order in which the Event was generated (within the scope of the Aggregate it originated from).

* Parameters assignable to Message will have the entire `EventMessage` injected \(if the message is assignable to that parameter\). If the first parameter is of type message, it effectively matches an Event of any type, even if generic parameters would suggest otherwise. Due to type erasure, Axon cannot detect what parameter is expected. In such case, it is best to declare a parameter of the payload type, followed by a parameter of type Message.

* When using Spring and the Axon Configuration is activated (either by including the Axon Spring Boot Starter module, or by specifying `@EnableAxon` on your `@Configuration` file), any other parameters will resolve to autowired beans, if exactly one injectable candidate is available in the application context. This allows you to inject resources directly into `@EventHandler` annotated methods.

You can configure additional `ParameterResolver`s by implementing the `ParameterResolverFactory` interface and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class. See [Advanced Customizations](../part4/advanced-customizations.md) for details.

In all circumstances, at most one event handler method is invoked per listener instance. Axon will search for the most specific method to invoke, using following rules:

1. On the actual instance level of the class hierarchy (as returned by `this.getClass()`), all annotated methods are evaluated

2. If one or more methods are found of which all parameters can be resolved to a value, the method with the most specific type is chosen and invoked

3. If no methods are found on this level of the class hierarchy, the super type is evaluated the same way

4. When the top level of the hierarchy is reached, and no suitable event handler is found, the event is ignored.

```java
// assume EventB extends EventA 
// and    EventC extends EventB

public class TopListener {

    @EventHandler
    public void handle(EventA event) {
    }

    @EventHandler
    public void handle(EventC event) {
    }
}

public class SubListener extends TopListener {

    @EventHandler
    public void handle(EventB event) {
    }

}
```

In the example above, the `SubListener` will receive all instances of `EventB` as well as `EventC` (as it extends `EventB`). In other words, the `TopListener` will not receive any invocations for `EventC` at all. Since `EventA` is not assignable to `EventB` (it's its superclass), those will be processed by `TopListener`.

Registering Event Handlers
-----------------------------
Event Handling components are defined using an `EventHandlingConfiguration` class, which is registered as a module with the global Axon `Configurer`. Typically, an application will have a single `EventHandlingConfiguration` defined, but larger more modular applications may choose to define one per module.

To register objects with `@EventHandler` methods, use the `registerEventHandler` method on the `EventHandlingConfiguration`:

```java
// define an EventHandlingConfiguration
EventHandlingConfiguration ehConfiguration = new EventHandlingConfiguration()
    .registerEventHandler(conf -> new MyEventHandlerClass());

// the module needs to be registered with the Axon Configuration
Configurer axonConfigurer = DefaultConfigurer.defaultConfiguration()
    .registerModule(ehConfiguration);
```

Event Processors
----------------
Event Handlers define the business logic to be performed when an Event is received. Event Processors are the components that take care of the technical aspects of that processing. They start a Unit of Work and possibly a transaction, but also ensure that correlation data can be correctly attached to all messages created during Event processing.

Event Processors come in roughly two forms: Subscribing and Tracking. he Subscribing processors subscribe themselves to a source of Events and are invoked by the thread managed by the publishing mechanism. Tracking Processors, on the other hand, pull their messages from a source using a thread that it manages itself.

### Assigning handlers to processors

All processors have a name, which identifies a processor instance across JVM instances. Two processors with the same name, can be considered as two instances of the same processor.

All Event Handlers are attached to a Processor which name is the package name of the Event Handler's class.

For example, the following classes:

- `org.axonframework.example.eventhandling.MyHandler`,
- `org.axonframework.example.eventhandling.MyOtherHandler`, and
- `org.axonframework.example.eventhandling.module.MyHandler`

will trigger the creation of two Processors:

- `org.axonframework.example.eventhandling` with 2 handlers, and 
- `org.axonframework.example.eventhandling.module` with a single handler

The Configuration API allows you to configure other strategies for assigning classes to processors, or even assign specific instances to specific processors.

### Configuring processors

By default, Axon will use Subscribing Event Processors. It is possible to change how Handlers are assigned and how processors are configured using the `EventHandlingConfiguration` class of the Configuration API.

The `EventHandlingConfiguration` class defines a number of methods that can be used to define how processors need to be configured.

- `registerEventProcessorFactory` allows you to define a default factory method that creates Event Processors for which no explicit factories have been defined.

- `registerEventProcessor(String name, EventProcessorBuilder builder)` defines the factory method to use to create a Processor with given `name`. Note that such Processor is only created if `name` is chosen as the processor for any of the available Event Handler beans.

- `registerTrackingProcessor(String name)` defines that a processor with given name should be configured as a Tracking Event Processor, using default settings. It is configured with a TransactionManager, TokenStore

- `usingTrackingProcessors()` sets the default to Tracking Processors instead of Subscribing ones.

Tracking Processors, unlike Subscribing ones, need a Token Store to store their progress in. Each message a Tracking Processor receives through its Event Stream is accompanies with a Token. This Token allows the processor to reopen the Stream at any later point, picking up where it left off with the last Event.

The Configuration API takes the Token Store, as well as most other components Processors need from the Global Configuration instance. If no TokenStore is explicitly defined, an `InMemoryTokenStore` is used, which is *not recommended in production*.
