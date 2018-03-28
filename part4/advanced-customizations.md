Advanced Customizations
=======================

Parameter Resolvers
-------------------

You can configure additional `ParameterResolver`s by extending the `ParameterResolverFactory` class and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class.

> **Caution**
>
> At this moment, OSGi support is limited to the fact that the required headers are mentioned in the manifest file. The automatic detection of `ParameterResolverFactory` instances works in OSGi, but due to classloader limitations, it might be necessary to copy the contents of the `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` file to the OSGi bundle containing the classes to resolve parameters for (i.e. the event handler).

Meta Annotations
----------------

TODO

Customizing Message Handler behavior
------------------------------------

HanderEnhancers
---------------

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
