# Spring AMQP

Spring AMQP is an alternative approach to distributing events, besides Axon Server which is default.

Axon provides out-of-the-box support to transfer events to and from an AMQP message broker, such as [RabbitMQ](https://www.rabbitmq.com/).

To use the Spring AMQP components from Axon, make sure the `axon-amqp` module is available on the classpath.

## Forwarding events to an AMQP Exchange

The `SpringAMQPPublisher` forwards events to an AMQP exchange. It is initialized with a `SubscribableMessageSource`, which is generally the `EventBus` or `EventStore`. Theoretically, this could be any source of events that the publisher can subscribe to.

To configure the \`SpringAMQPPublisher\`, simply define an instance as a Spring Bean. There is a number of setter methods that allow you to specify the behavior you expect, such as transaction support, publisher acknowledgements \(if supported by the broker\), and the exchange name.

The default exchange name is `"Axon.EventBus"`.

> **Note**
>
> Note that exchanges are not automatically created. You must still declare the Queues, Exchanges and Bindings you wish to use. Check the Spring documentation for more information.

## Reading events from an AMQP Queue

Spring has extensive support for reading messages from an AMQP Queue. However, this needs to be 'bridged' to Axon, so that these messages can be handled from Axon as if they are regular event messages.

The `SpringAMQPMessageSource` allows event processors to read messages from a queue, instead of the event store or event bus. It acts as an adapter between Spring AMQP and the `SubscribableMessageSource` needed by these processors.

The easiest way to configure the `SpringAMQPMessageSource`, is by defining a bean which overrides the default `onMessage` method and annotates it with `@RabbitListener`, as follows:

```java
@Bean
public SpringAMQPMessageSource myMessageSource(Serializer serializer) {
    return new SpringAMQPMessageSource(serializer) {
        @RabbitListener(queues = "myQueue")
        @Override
        public void onMessage(Message message, Channel channel) throws Exception {
            super.onMessage(message, channel);
        }
    };
}
```

Spring its `@RabbitListener` annotation tells Spring that this method needs to be invoked for each message on the given queue \(`'myQueue'` in the example\). This method simply invokes the `super.onMessage()` method, which performs the actual publication of the Event to all the processors that have been subscribed to it.

To subscribe processors to this `MessageSource`, pass the correct `SpringAMQPMessageSource` instance to the constructor of the subscribing event processor:

```java
// in an @Configuration file
@Autowired
public void configure(EventProcessingConfigurer epConfig, SpringAmqpMessageSource myMessageSource) {
    epConfig.registerSubscribingEventProcessor("myProcessor", c -> myMessageSource);
}
```

Note that tracking processors are not compatible with the `SpringAMQPMessageSource`.
