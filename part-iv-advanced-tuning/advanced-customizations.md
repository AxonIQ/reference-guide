# Advanced customizations

## Parameter Resolvers

You can configure additional `ParameterResolver`s by extending the `ParameterResolverFactory` class and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class.

> **Caution**
>
> At this moment, OSGi support is limited to the fact that the required headers are mentioned in the manifest file. The automatic detection of `ParameterResolverFactory` instances works in OSGi, but due to classloader limitations, it might be necessary to copy the contents of the `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` file to the OSGi bundle containing the classes to resolve parameters for \(i.e. the event handler\).

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

By specifying the `payloadType` on the `@CommandHandler` meta-annotation, this becomes the value used for all Command Handlers annotated with `JsonCommandHandler`. These command handlers may (and should) still provide a parameter for the payload, but Axon will complain if it isn't a subclass of `JsonNode`.

The `commandName` attribute on the `JsonCommandHandler` annotation does not have a default value, and will therefore force developers to specify the name of the command. Note that, to override values, the attribute name must identical to the name on the `@CommandHandler` meta-annotation.

Lastly, the `routingKey` property is defined exactly as in the `@CommandHandler` annotation's specification to still allow developers to choose to provide a Routing Key when using the `JsonCommandHandler`.

When writing custom logic to access properties of annotation that may be meta-annotated, be use to use the `AnnotationUtils#findAnnotationAttributes(AnnotatedElement, String)` method, or the `annotationAttributes` on the `MessageHandlingMember`. Using Java's annotation API will not consider meta-annotations.

## Customizing Message Handler behavior

Overriding annotations is very useful to implement best practices that you have established in your team, providing defaults or restrictions of how annotations may be used. However, they can also be very useful when special behavior needs to be added to message handlers based on the presence of an annotation.

### HandlerEnhancers

Handler Enhancers allow you to wrap handlers and add custom logic to the execution, or eligibility of handlers for a certain message. This differs from HandlerInterceptors in that you have access to the Aggregate member at the time of resolving, and it allows for more fine-grained control
.
You can use HandlerEnhancers to intercept and perform checks on groups of `@CommandHandler`s or `@EventHandler`s. 

To create a HandlerEnhancer you start by implementing `HandlerEnhancerDefinition` and overriding the `wrapHandler()` method. 
All this method does is give you access to the `MessageHandlingMember<T>` which is an object representing any handler that is specified in the system. 

You can then sort these handlers based on their annotations by using the `annotationAttributes(Annotation annotation)` method. This will filter out only those handlers that use that `Annotation`.

For your HandlerEnhancer to run you'll need to create a `META-INF/services/org.axonframework.messaging.annotation.HandlerEnhancerDefinition` file containing
the fully qualified class name of the handler enhancer you have created.

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

