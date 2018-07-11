# Advanced customizations

## Parameter Resolvers

You can configure additional `ParameterResolver`s by extending the `ParameterResolverFactory` class and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class.

> **Caution**
>
> At this moment, OSGi support is limited to the fact that the required headers are mentioned in the manifest file. The automatic detection of `ParameterResolverFactory` instances works in OSGi, but due to classloader limitations, it might be necessary to copy the contents of the `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` file to the OSGi bundle containing the classes to resolve parameters for \(i.e. the event handler\).

## Serializers

Serializers come in several flavors in the Axon Framework and are used for a variety of subjects. Currently you can choose between the `XStreamSerializer`, `JacksonSerializer` and `JavaSerializer` to serialize the messages (commands/queries/events), tokens, snapshots and sagas in an Axon application. 

As there are several objects to be serialized, it is typically desired to tune which serializer handles which. To that end, the `Configuration` API allows you to define a default, message and event serializer, which lead to the following object-serialization break down:

1. The Event `Serializer` is in charge of de-/serializing Event messages.
2. The Message `Serializer` is in charge of de-/serializing the Command and Query messages (used in a distributed application set up). 
3. The Default `Serializer` is in charge of de-/serializing the remainder, being the Tokens, Snapshots and Sagas.

By default all three `Serializer` flavors are set to use the `XStreamSerializer`, which internally uses the [XStream](http://x-stream.github.io/) to serialize objects to an XML format. XML is a verbose format to serialize to, but has the major benefit of being able to serialize almost everything. This verbosity is typically fine when storing your tokens, sagas and snapshots, but for messages (and specifically events) XML might proof to cost to much due to its serialized size. Thus for optimization reasons you can configure different serializers for your messages. 

There is an implicit ordering between the configurable serializer. If no Event `Serializer` is configured, the Event de-/serialization will be performed by the Message `Serializer`. In turn, if not Message `Serializer` is configured, the Default `Serializer` will take that role.

See the following example on how to configure each serializer specifically, were we use `XStreamSerializer` as the default and `JacksonSerializer` for all our messages: 

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

## Meta Annotations

TODO

## Customizing Message Handler behavior

### HanderEnhancers

It is possible to wrap handlers with custom logic. This differs from HandlerInterceptors in that you have access to the Aggregate member at the time of resolving.
You can use HandlerEnhancers to intercept and perform checks on groups of `@CommandHandler`s or `@EventHandler`s. 

To create a HandlerEnhancer you start by implementing `HandlerEnhancerDefinition` and overriding the `wrapHandler()` method. 
All this method does is give you access to the `MessageHandlingMember<T>` which is an object representing any handler that is specified in the system. 

You can then sort these handlers based on their annotations by using the `annotationAttributes(Annotation annotation)` method. This will filter out only those handlers that use that `Annotation`.

For your HandlerEnhancer to run you'll need to create a `META-INF/services/org.axonframework.messaging.annotation.HandlerEnhancerDefinition` file containing
the fully qualified name of the handler enhancer you would like to use.

Example:

```java
public class MethodCommandHandlerDefinition implements HandlerEnhancerDefinition { // 1.

    @Override
    public <T> MessageHandlingMember<T> wrapHandler(MessageHandlingMember<T> original) { // 2.
        return original.annotationAttributes(CommandHandler.class) // 3.
                .map(attr -> (MessageHandlingMember<T>) new MethodCommandMessageHandlingMember(original, attr))
                .orElse(original); // 5.
    }

    private static class MethodCommandMessageHandlingMember<T> extends WrappedMessageHandlingMember<T>{

        private final String commandName;

        private MethodCommandMessageHandlingMember(MessageHandlingMember<T> delegate,
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
            return super.canHandle(message) && commandName.equals(((CommandMessage) message).getCommandName()); // 4.
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

