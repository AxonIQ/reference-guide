# 1.2.2 Event handling

Event listeners are the components that act on incoming events. They typically execute logic based on decisions that have been made by the command model. Usually, this involves updating view models or forwarding updates to other components, such as third party integrations. In some cases event handlers will throw events themselves based on \(patterns of\) events that they received, or even send commands to trigger further changes.

## Defining event handlers

In Axon, an object may declare a number of event handler methods, by annotating them with `@EventHandler`. The declared parameters of the method define which events it will receive.

Axon provides out-of-the-box support for the following parameter types:

* The first parameter is always the payload of the event message. In the case the event handlers does not need access to the payload of the message, you can specify the expected payload type on the `@EventHandler` annotation. When specified, the first parameter is resolved using the rules specified below. Do not configure the payload type on the annotation if you want the payload to be passed as a parameter.
* Parameters annotated with `@MetaDataValue` will resolve to the metadata value with the key as indicated on the annotation. If `required` is `false` \(default\), `null` is passed when the metadata value is not present. If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.
* Parameters of type `MetaData` will have the entire `MetaData` of an `EventMessage` injected.
* Parameters annotated with `@Timestamp` and of type `java.time.Instant` \(or `java.time.temporal.Temporal`\) will resolve to the timestamp of the `EventMessage`. This is the time at which the event was generated.
* Parameters annotated with `@SequenceNumber` and of type `java.lang.Long` or `long` will resolve to the `sequenceNumber` of a `DomainEventMessage`. This provides the order in which the event was generated \(within the scope of the aggregate it originated from\).
* Parameters assignable to message will have the entire `EventMessage` injected \(if the message is assignable to that parameter\). If the first parameter is of type message, it effectively matches an event of any type, even if generic parameters would suggest otherwise. Due to type erasure, Axon cannot detect what parameter is expected. In such case, it is best to declare a parameter of the payload type, followed by a parameter of type message.
* When using Spring and the Axon Configuration is activated \(either by including the Axon Spring Boot Starter module, or by specifying `@EnableAxon` on your `@Configuration` file\), any other parameters will resolve to autowired beans, if exactly one injectable candidate is available in the application context. This allows you to inject resources directly into `@EventHandler` annotated methods.

You can configure additional `ParameterResolver`s by implementing the `ParameterResolverFactory` interface and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class. See [Advanced Customizations](../1.4-advanced-tuning/advanced-customizations.md) for details.

In all circumstances, at most one event handler method is invoked per listener instance. Axon will search for the most specific method to invoke, using following rules:

1. On the actual instance level of the class hierarchy \(as returned by `this.getClass()`\), all annotated methods are evaluated
2. If one or more methods are found of which all parameters can be resolved to a value, the method with the most specific type is chosen and invoked
3. If no methods are found on this level of the class hierarchy, the super type is evaluated the same way
4. When the top level of the hierarchy is reached, and no suitable event handler is found, the event is ignored.

```java
// assume EventB extends EventA 
// and    EventC extends EventB
// and    a single instance of SubListener is registered

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

In the example above, the handler methods of `SubListener` will be invoked for all instances of `EventB` as well as `EventC` \(as it extends `EventB`\). In other words, the handler methods of `TopListener` will not receive any invocations for `EventC` at all. Since `EventA` is not assignable to `EventB` \(it is its superclass\), those will be processed by the handler method in `TopListener`.

## Registering event handlers

Event handling components are defined using an `EventProcessingConfigurer`, which can be accessed from the global Axon 
`Configurer`. `EventProcessingConfigurer` is used to configure an `EventProcessingConfiguration`. Typically, an 
application will have a single `EventProcessingConfiguration` defined, but larger more modular applications may choose 
to define one per module.

To register objects with `@EventHandler` methods, use the `registerEventHandler()` method on the `EventProcessingConfigurer`:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
                                         .eventProcessing(eventProcessingConfigurer -> eventProcessingConfigurer
                                             .registerEventHandler(conf -> new MyEventHandlerClass()));
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Component
public class MyEventHandlerClass {
    // contains @EventHandler(s)
}
```
{% endtab %}
{% endtabs %}

See [Event handling configuration](../1.3-infrastructure-components/spring-boot-autoconfiguration.md#event-processing-configuration) for details on registering event handlers using Spring AutoConfiguration.

