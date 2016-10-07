Event Listeners
===============

TODO: Update to Axon 3

Event listeners are the components that act on incoming events. Events may be of any type. In the Axon Framework, all event listeners must implement the `EventListener` interface.

## Annotated Event Handler

Implementing the `EventListener` interface can produce a large if-statement and verbose plumbing code. Using annotations to demarcate Event Handler methods is a cleaner alternative.

The `AnnotationEventListenerAdapter` can wrap any object into an event listener. The adapter will invoke the most appropriate event handler method available. These event handler methods must be annotated with the `@EventHandler` annotation.

The `AnnotationEventListenerAdapter`, as well as the `AbstractAnnotatedAggregateRoot`, use `ParameterResolver`s to resolve the value that should be passed in the parameters of methods annotated with `@EventHandler`. The `AnnotationEventListenerAdapter` uses a `ParameterResolverFactory` to resolve the values for the parameters of the annotated methods. By default, Axon uses a `ClasspathParameterResolverFactory`, that looks for `ParameterResolverFactory` instances using the ServiceLoader mechanism.

Axon provides out-of-the-box support for the following parameter types:

* The first parameter is always the payload of the Event message. In the case the Event Handlers doesn't need access to the payload of the message, you can specify the expected payload type on the `@EventHandler` annotation. When specified, the first parameter is resolved using the rules specified below. Do not configure the payload type on the annotation if you want the payload to be passed as a parameter.

* Parameters annotated with `@MetaData` will resolve to the Meta Data value with the key as indicated on the annotation. If `required` is `false` \(default\), `null` is passed when the meta data value is not present. If `required` is `true`, the resolver will not match and prevent the method from being invoked when the meta data value is not present.

* Parameters of type `MetaData` will have the entire `MetaData` of an `EventMessage` injected.

* Parameters annotated with `@Timestamp` and of type `org.joda.time.DateTime` will resolve to the timestamp of the `EventMessage`. This is the time at which the Event was generated.

* Parameters annotated with `@SequenceNumber` and of type `java.lang.Long` or `long` will resolve to the `sequenceNumber` of a `DomainEventMessage`. This provides the order in which the Event was generated.

* Parameters assignable to Message will have the entire `EventMessage` injected \(if the message is assignable to that parameter\). If the first parameter is of type message, it effectively matches an Event of any type, even if generic parameters would suggest otherwise. Due to type erasure, Axon cannot detect what parameter is expected. In such case, it is best to declare a parameter of the payload type, followed by a parameter of type Message.

* When using Spring and `<axon:annotation-config/>` is declared, any other parameters will resolve to autowired beans, if exactly one injectable candidate is available in the application context. This allows you to inject resources directly into `@EventHandler` annotated methods. Note that this only works for Spring-managed beans. Event Sourcing handlers on Aggregate instances don't get resources injected.


You can configure additional `ParameterResolver`s by implementing the `ParameterResolverFactory` interface and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class.

> **Caution**
> 
> At this moment, OSGi support is limited to the fact that the required headers are mentioned in the manifest file. The automatic detection of ParameterResolverFactory instances works in OSGi, but due to classloader limitations, it might be necessary to copy the contents of the `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` file to the OSGi bundle containing the classes to resolve parameters for (e.g. the event handlers).

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

In the example above, the `SubListener` will receive all instances of `EventB` as well as `EventC` \(as it extends `EventB`\). In other words, the `TopListener` will not receive any invocations for `EventC` at all. Since `EventA` is not assignable to `EventB` \(it's its superclass\), those will be processed by `TopListener`.

The constructor of the `AnnotationEventListenerAdapter` takes two parameters: the annotated bean, and optionally a `ParameterResolverFactory`, which defines how parameters are resolved, defaulting to the mechanism described above. The adapter instance can then be registered to the Event Bus using the `subscribe()` method on the `EventBus`.

> **Tip**
> 
> If you use Spring, you can automatically wrap all annotated event listeners with an adapter automatically by adding `<axon:annotation-config/>` to your application context. Axon will automatically find and wrap annotated event listeners in the Application Context with an `AnnotationEventListenerAdapter` and register them with the Event Bus.
