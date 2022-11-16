# Handler Enhancers

Handler Enhancers allow you to wrap handlers and add custom logic to the execution, or eligibility of handlers for a certain message. 
This differs from `HandlerInterceptor`s in that you have access to the aggregate member at the time of resolving,
 and it allows for more fine-grained control. 
You can use handler enhancers to intercept and perform checks on groups of `@CommandHandler`s or `@EventHandler`s.

To create a handler enhancer you start by implementing `HandlerEnhancerDefinition` and overriding the `wrapHandler()` method. 
All this method does is give you access to the `MessageHandlingMember<T>` which is an object representing any handler that is specified in the system.

You can then sort these handlers based on their annotations by using the `annotationAttributes(Annotation annotation)` method. 
This will filter out only those handlers that use that `Annotation`.

For your handler enhancer to run you'll need to either create a `META-INF/services/org.axonframework.messaging.annotation.HandlerEnhancerDefinition` file containing the fully qualified class name of the handler enhancer you have created, or register it explicitly in the Configurer.

Example of a Handler Enhancer that filters messages based on an expected Meta-Data key and value.

```java
// 1
public class ExampleHandlerDefinition implements HandlerEnhancerDefinition {

    // 2
    @Override
    public <T> MessageHandlingMember<T> wrapHandler(MessageHandlingMember<T> original) {
        return original.attribute("metaDataKey") // 3
                       .map(attr -> new ExampleMessageHandlingMember<>(original))
                       .map(member -> (MessageHandlingMember<T>) member)
                       .orElse(original); // 6
    }

    private static class ExampleMessageHandlingMember<T> extends WrappedMessageHandlingMember<T> {

        private final String metaDataKey;
        private final String expectedValue;

        private ExampleMessageHandlingMember(MessageHandlingMember<T> delegate) {
            super(delegate);
            metaDataKey = (String) delegate.attribute("metaDataKey")
                                           .orElseThrow(() -> new IllegalArgumentException("Missing expected attribute"));
            expectedValue = (String) delegate.attribute("expectedValue")
                                             .orElseThrow(() -> new IllegalArgumentException("Missing expected attribute"));
        }

        @Override
        public boolean canHandle(Message<?> message) {
            // 4
            return super.canHandle(message) && expectedValue.equals(message.getMetaData().get(metaDataKey));
        }
    }
}

// ...

// 5
@HasHandlerAttributes
public @interface MyAnnotation {

    String metaDataKey();

    String expectedValue();
}
```

1. Implement the `HandlerEnhancerDefinition` interface
2. Override the `wrapHandler` method to perform your own logic.
3. Sort out the types of handlers you want to wrap based on a specific attribute, for example, the `metaDataKey` attribute from the `MyAnnotation`.
4. Handle the method inside of a `MessageHandlingMember`, in this case, indicating the handler is only suitable if the meta-data key matches a value.
5. For annotation-specific attributes to exist in the `MessageHandlingMember's` attribute collection, meta-annotation the custom annotation with `HasHandlerAttributes`.
6. If you are not interested in wrapping the handler, just return the original that was passed into the `wrapHandler` method.

It is possible to configure an `HandlerDefinition` with Axon `Configuration`. 
If you are using Spring Boot defining `HandlerDefintion`s and `HandlerEnhancerDefinition`s as beans is sufficient \(Axon autoconfiguration will pick them up and configure within Axon `Configuration`\).

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
@Configuration
public class AxonConfig { 
    // omitting other configuration methods...
    public void registerHandlerDefinition(Configurer configurer) {
        configurer.registerHandlerDefinition((c, clazz) -> MultiHandlerDefinition.ordered(
                MultiHandlerEnhancerDefinition.ordered(
                        ClasspathHandlerEnhancerDefinition.forClass(clazz), 
                        new CustomHandlerEnhancerDefinition()
                ), 
                new CustomHandlerDefinition(), 
                ClasspathHandlerDefinition.forClass(clazz)
        ));
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Configuration
public class AxonConfig {
 // omitting other (bean) configuration methods...
 @Bean
 public HandlerDefinition customHandlerEnhancer() {
  return new CustomHandlerDefinition();
 }

 @Bean
 public HandlerEnhancerDefinition customHandlerEnhancerDefinition() {
  return new CustomHandlerEnhancerDefinition();
 }
}
```
{% endtab %}
{% endtabs %}

