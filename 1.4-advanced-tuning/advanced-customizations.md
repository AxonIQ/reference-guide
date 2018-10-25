# 1.4.1 Advanced customizations

## Parameter Resolvers

You can configure additional `ParameterResolver`s by extending the `ParameterResolverFactory` class and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class.

> **Caution**
>
> At this moment, OSGi support is limited to the fact that the required headers are mentioned in the manifest file. The automatic detection of `ParameterResolverFactory` instances works in OSGi, but due to classloader limitations, it might be necessary to copy the contents of the `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` file to the OSGi bundle containing the classes to resolve parameters for \(i.e. the event handler\).

## Serializers

Serializers come in several flavors in the Axon Framework and are used for a variety of subjects. Currently you can choose between the `XStreamSerializer`, `JacksonSerializer` and `JavaSerializer` to serialize the messages \(commands/queries/events\), tokens, snapshots and sagas in an Axon application.

As there are several objects to be serialized, it is typically desired to tune which serializer handles which. To that end, the `Configuration` API allows you to define a default, message and event serializer, which lead to the following object-serialization break down:

1. The `EventSerializer` is in charge of de-/serializing event messages. Events are typically stored in an event store for a long period of time. This is the main driver for choosing the event serializer implementation.
2. The `MessageSerializer` is in charge of de-/serializing the command and query messages \(used in a distributed application set up\). Messages are shared between nodes and typically need to be interoperable and/or compact. Take this into account when choosing the message serializer. 
3. The default`Serializer` is in charge of de-/serializing the remainder, being the tokens, snapshots and sagas. These objects are generally not shared between different applications, and most of these classes aren't expected to have some of the getters and setters that are, for example, typically required by Jackson based serializers. A flexible, general purpose serializer like [XStream](http://x-stream.github.io/) is quite suited for this purpose.

By default all three `Serializer` flavors are set to use the `XStreamSerializer`, which internally uses [XStream](http://x-stream.github.io/) to serialize objects to an XML format. XML is a verbose format to serialize to, but XStream has the major benefit of being able to serialize virtually anything. This verbosity is typically fine when storing tokens, sagas or snapshots, but for messages \(and specifically events\) XML might prove to cost too much due to its serialized size. Thus for optimization reasons you can configure different serializers for your messages. Another very valid reason for customizing Serializers is to achieve interoperability between different \(Axon\) applications, where the receiving end potentially enforces a specific serialized format.

There is an implicit ordering between the configurable serializer. If no `EventSerializer` is configured, the Event de-/serialization will be performed by the `MessageSerializer`. In turn, if no `MessageSerializer` is configured, the default `Serializer` will take that role.

See the following example on how to configure each serializer specifically, were we use `XStreamSerializer` as the default and `JacksonSerializer` for all our messages:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class SerializerConfiguration {

    public void serializerConfiguration(Configurer configurer) {
        // Per default we want the XStreamSerializer
        XStreamSerializer defaultSerializer = new XStreamSerializer();
        // But for all our messages we'd prefer the JacksonSerializer due to JSON its smaller format
        JacksonSerializer messageSerializer = new JacksonSerializer();

        configurer.configureSerializer(configuration -> defaultSerializer)
                  .configureMessageSerializer(configuration -> messageSerializer)
                  .configureEventSerializer(configuration -> messageSerializer);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties file" %}
```text
# Possible values for these keys are `default`, `xstream`, `java`, and `jackson`.
axon.serializer.general
axon.serializer.events
axon.serializer.messages
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - YML file" %}
```yaml
# Possible values for these keys are `default`, `xstream`, `java`, and `jackson`.
axon:
    serializer:
        general: 
        events: 
        messages:
```
{% endtab %}
{% endtabs %}

## Meta Annotations

Most annotations in Axon can be placed on other annotations, as so-called meta-annotation. When Axon scans for annotations, it will automatically scan meta-annotations as well. Annotations can override the properties defined on the meta-annotations, if desired.

For example, if you have a practice in your development team that payloads are always represented in JSON and you wish the command name to be explicitly configured, you could create your own annotation:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.CONSTRUCTOR, ElementType.ANNOTATION_TYPE})
@CommandHandler(payloadType = JsonNode.class)
public @interface JsonCommandHandler {

    String commandName;

    String routingKey() default "";
}
```

By specifying the `payloadType` on the `@CommandHandler` meta-annotation, this becomes the value used for all Command Handlers annotated with `JsonCommandHandler`. These command handlers may \(and should\) still provide a parameter for the payload, but Axon will complain if it isn't a subclass of `JsonNode`.

The `commandName` attribute on the `JsonCommandHandler` annotation does not have a default value, and will therefore force developers to specify the name of the command. Note that, to override values, the attribute name must identical to the name on the `@CommandHandler` meta-annotation.

Lastly, the `routingKey` property is defined exactly as in the `@CommandHandler` annotation's specification to still allow developers to choose to provide a Routing Key when using the `JsonCommandHandler`.

When writing custom logic to access properties of annotation that may be meta-annotated, be sure to use the `AnnotationUtils#findAnnotationAttributes(AnnotatedElement, String)` method, or the `annotationAttributes` on the `MessageHandlingMember`. Using Java's annotation API will not take meta-annotations into consideration.

## Customizing Message Handler behavior

Overriding annotations is very useful to implement best practices that you have established in your team, providing defaults or restrictions of how annotations may be used. However, they can also be very useful when special behavior needs to be added to message handlers based on the presence of an annotation.

### Handler Enhancers

Handler Enhancers allow you to wrap handlers and add custom logic to the execution, or eligibility of handlers for a certain message. This differs from `HandlerInterceptor`s in that you have access to the Aggregate member at the time of resolving, and it allows for more fine-grained control. You can use Handler Enhancers to intercept and perform checks on groups of `@CommandHandler`s or `@EventHandler`s.

To create a Handler Enhancer you start by implementing `HandlerEnhancerDefinition` and overriding the `wrapHandler()` method. All this method does is give you access to the `MessageHandlingMember<T>` which is an object representing any handler that is specified in the system.

You can then sort these handlers based on their annotations by using the `annotationAttributes(Annotation annotation)` method. This will filter out only those handlers that use that `Annotation`.

For your HandlerEnhancer to run you'll need to create a `META-INF/services/org.axonframework.messaging.annotation.HandlerEnhancerDefinition` file containing the fully qualified class name of the handler enhancer you have created.

Example:

```java
// 1
public class MethodCommandHandlerDefinition implements HandlerEnhancerDefinition { 

    @Override // 2
    public <T> MessageHandlingMember<T> wrapHandler(MessageHandlingMember<T> original) {
        return original.annotationAttributes(CommandHandler.class) // 3
                .map(attr -> (MessageHandlingMember<T>) 
                             new MethodCommandMessageHandlingMember(original, attr))
                .orElse(original); // 5
    }

    private static class MethodCommandMessageHandlingMember<T> 
                             extends WrappedMessageHandlingMember<T>{

        private final String commandName;

        private MethodCommandMessageHandlingMember(
                             MessageHandlingMember<T> delegate,
                             Map<String, Object> annotationAttributes) {
            super(delegate);

            if ("".equals(annotationAttributes.get("commandName"))) {
                commandName = delegate.payloadType().getName();
            } else {
                commandName = (String) annotationAttributes.get("commandName");
            }
        }

        @Override
        public boolean canHandle(Message<?> message) {
            return super.canHandle(message) && commandName.equals(
                             ((CommandMessage) message).getCommandName()); // 4
        }

        public String commandName() {
            return commandName;
        }
    }
}
```

1. Implement the `HandlerEnhancerDefinition` interface
2. Override the `wrapHandler` method to perform your own logic.
3. Sort out the types of messages you want to handle, for example any `CommandHandler`s or your own custom annotation even.
4. Handle the method inside of a `MessageHandlingMember`
5. If you would like to skip handling just return the original that was passed into the `wrapHandler` method.

To skip all handling of the handler then just throw an exception.

It is possible to configure `HandlerDefinition` with Axon `Configuration`. If you are using Spring Boot defining `HandlerDefintion`s and `HandlerEnhancerDefinition`s as beans is sufficient \(Axon autoconfiguration will pick them up and configure within Axon `Configuration`\).

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.registerHandlerDefinition(c -> new MethodCommandHandlerDefinition());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
// somewhere in configuration
@Bean
public HandlerDefinition eventStorageEngine() {
    return new MethodCommandHandlerDefinition(); 
}
```
{% endtab %}
{% endtabs %}

## Filtering Event Storage Engine

In some scenarios you might want to chose which events to store. For this you can use the `FilteringEventStorageEngine`. One imaginable use case could be that we don't want to store non-domain events. `FilteringEventStorageEngine` uses a `Predicate<? super EventMessage<?>>` in order to filter events which get stored in the delegated engine. Let's try to configure a `FilteringEventStorageEngine` with the `Configurer` \(if you are using Spring, it's enough to have a bean of type `EventStorageEngine` in your `ApplicationContext`\). The next example will only store domain events:

```java
public class EventStorageEngineConfiguration {

    public void configureEventStorageEngine(Configurer configurer) {
        // It does not matter for this example what storage engine you use
        EventStorageEngine delegate = ...; 

        configurer.configureEmbeddedEventStore(c -> 
            new FilteringEventStorageEngine(delegate, 
                                     em -> em instanceof DomainEventMessage));       
    }
}
```

