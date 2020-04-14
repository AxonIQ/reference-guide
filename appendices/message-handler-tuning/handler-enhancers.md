# Handler Enhancers

Handler Enhancers allow you to wrap handlers and add custom logic to the execution, or eligibility of handlers for a certain message. This differs from `HandlerInterceptor`s in that you have access to the aggregate member at the time of resolving, and it allows for more fine-grained control. You can use handler enhancers to intercept and perform checks on groups of `@CommandHandler`s or `@EventHandler`s.

To create a handler enhancer you start by implementing `HandlerEnhancerDefinition` and overriding the `wrapHandler()` method. All this method does is give you access to the `MessageHandlingMember<T>` which is an object representing any handler that is specified in the system.

You can then sort these handlers based on their annotations by using the `annotationAttributes(Annotation annotation)` method. This will filter out only those handlers that use that `Annotation`.

For your handler enhancer to run you'll need to either create a `META-INF/services/org.axonframework.messaging.annotation.HandlerEnhancerDefinition` file containing the fully qualified class name of the handler enhancer you have created, or register it explicitly in the Configurer.

Example of a Handler Enhancer that filters messages based on an expected Meta-Data key and value.

```java
// 1
public class ExampleHandlerDefinition implements HandlerEnhancerDefinition { 

    @Override // 2
    public <T> MessageHandlingMember<T> wrapHandler(MessageHandlingMember<T> original) {
        return original.annotationAttributes(MyAnnotation.class) // 3
                .map(attr -> (MessageHandlingMember<T>) 
                             new ExampleMessageHandlingMember<>(original, attr))
                .orElse(original); // 5
    }

    private static class ExampleMessageHandlingMember<T> 
                             extends WrappedMessageHandlingMember<T>{

        private final String metaDataKey;
        private final String expectedValue;

        private ExampleMessageHandlingMember(
                             MessageHandlingMember<T> delegate,
                             Map<String, Object> annotationAttributes) {
            super(delegate);
            metaDataKey = (String) annotationAttributes.get("metaDataKey");
            expectedValue = (String) annotationAttributes.get("expectedValue");
        }

        @Override
        public boolean canHandle(Message<?> message) {
            return super.canHandle(message) && expectedValue.equals(message.getMetaData().get(metaDataKey)); // 4
        }
    }
}
```

1. Implement the `HandlerEnhancerDefinition` interface
2. Override the `wrapHandler` method to perform your own logic.
3. Sort out the types of handlers you want to wrap, for example any handlers with a `MyAnnotation`.
4. Handle the method inside of a `MessageHandlingMember`, in this case, indicating the handler is only suitable if the meta-data key matches a value.
5. If you are not interested in wrapping the handler, just return the original that was passed into the `wrapHandler` method.

It is possible to configure `HandlerDefinition` with Axon `Configuration`. If you are using Spring Boot defining `HandlerDefintion`s and `HandlerEnhancerDefinition`s as beans is sufficient \(Axon autoconfiguration will pick them up and configure within Axon `Configuration`\).

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.registerHandlerDefinition((c, clazz) ->
                                             MultiHandlerDefinition.ordered(
                                                     MultiHandlerEnhancerDefinition.ordered(
                                                             ClasspathHandlerEnhancerDefinition.forClass(clazz),
                                                             new MyCustomEnhancerDefinition()
                                                     ),
                                                     new MyCustomHandlerDefinition(),
                                                     ClasspathHandlerDefinition.forClass(clazz)
                                             ));
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
// somewhere in configuration
// for a HandlerDefinition
@Bean
public HandlerDefinition myCustomHandlerDefinition() {
    return new CustomHandlerDefinition(); 
}

// or to define a handler enhancer
@Bean
public HandlerEnhancerDefinition myCustomHandlerEnhancerDefinition() {
    return new MyCustomEnhancerDefinition(); 
}
```
{% endtab %}
{% endtabs %}

