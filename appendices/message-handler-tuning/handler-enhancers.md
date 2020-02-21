# Handler Enhancers 

Handler Enhancers allow you to wrap handlers and add custom logic to the execution, or eligibility of handlers for a certain message. This differs from `HandlerInterceptor`s in that you have access to the aggregate member at the time of resolving, and it allows for more fine-grained control. You can use handler enhancers to intercept and perform checks on groups of `@CommandHandler`s or `@EventHandler`s.

To create a handler enhancer you start by implementing `HandlerEnhancerDefinition` and overriding the `wrapHandler()` method. All this method does is give you access to the `MessageHandlingMember<T>` which is an object representing any handler that is specified in the system.

You can then sort these handlers based on their annotations by using the `annotationAttributes(Annotation annotation)` method. This will filter out only those handlers that use that `Annotation`.

For your handler enhancer to run you'll need to create a `META-INF/services/org.axonframework.messaging.annotation.HandlerEnhancerDefinition` file containing the fully qualified class name of the handler enhancer you have created.

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
