# Apache Kafka

Kafka is an alternative approach to distributing events, besides Axon Server which is default.

Kafka is a very popular system for publishing and consuming events. It's architecture is fundamentally different from most messaging systems, and combines speed with reliability.

To use the Kafka components from Axon, make sure the `axon-kafka` module is available on the classpath.

{% hint style="info" %}
The `axon-kafka` module is a new addition to the framework. Minor releases of the framework could include breaking changes to the APIs.
{% endhint %}

## Publishing Events to a Kafka topic

When Event Messages are published to an Event Bus \(or Event Store\), they can be forwarded to a Kafka topic using the `KafkaPublisher`. Publication of the messages to Kafka will happen in the same thread \(and Unit of Work\) that published the events to the Event Bus.

The `KafkaPublisher` takes a `KafkaPublisherConfiguration` instance, which provides the different values and settings required to publish events to Kafka.

```java
KafkaPublisherConfiguration configuration = KafkaPublisherConfiguration.<String, byte[]>builder()
        .withProducerFactory(factory) // the factory for creating the actual client instances for sending events to kafka
        .withTopic("topic") // the topic to send the events to. Defaults to 'Axon.Events'
        .build();

KafkaPublisher<String, byte[]> publisher = new KafkaPublisher<>(configuration); // create the publisher itself

publisher.start(); // to start publishing all events
```

Axon provides a `DefaultProducerFactory`, which attempts to reuse created instances to avoid continuous creation of new ones. It's creation uses a similar builder pattern. The builder requires a `configs` Map, which are the settings to use for the Kafka client, such as the Kafka instance locations. Please check the Kafka guide for the possible settings and their values.

```java
DefaultProducerFactory.builder(configs)
        .withConfirmationMode(ConfirmationMode.WAIT_FOR_ACK) // either TRANSACTIONAL, WAIT_FOR_ACK or NONE (default)
        .build();

// or, to create a transactional ProducerFactory
DefaultProducerFactory.builder(configs)
        .withTransactionalIdPrefix("myTxPrefix") // this will also set ConfirmationMode to TRANSACTIONAL
        .build();
```

Note that the `DefaultProducerFactory` needs to be `shutDown` properly, to ensure all producer instances are properly closed.

## Consuming Events from a Kafka topic

Messages can be consumed by Tracking Event Processors by configuring a `KafkaMessageSource`. This message source uses a `Fetcher` to retrieve the actual messages from Kafka. You can either use the `AsyncFetcher`, or provide your own.

The `AsyncFetcher` is initialized using a builder, which requires the Kafka Configuration to initialize the client. Please check the Kafka guide for the possible settings and their values.

```java
// the fetcher only requires Kafka Client Configuration properties:
AsyncFetcher.builder(configs).build();

// but customization is possible:
AsyncFetcher.builder(configs)
        .withTopic("myTopic") // the Kafka topic to read from. Defaults to 'Axon.Events'
        .withPool(customThreadPool) // defaults to a cached thread pool
        .withPollTimeout(customTimeout, timeUnit) // defaults to 5 seconds
        .onRecordPublished(callback) // register behavior to execute on every incoming message
        .withBufferFactory(bufferFactory) // to customize the implementation of the buffers used to hold messages before they are consumed
        .build();
```

The `AsyncFetcher` doesn't need to be explicitly started, as it will start when the first processors connect to it. It does need to be shut down, to ensure any thread pool or active connections are properly closed.

## Customizing message format

By default, Axon uses the `DefaultKafkaMessageConverter` to convert an `EventMessage` to a Kafka `ProducerRecord` and an `ConsumerRecord` back into an `EventMessage`. This implementation already allows for some customization, such as how the `Message`'s `MetaData` is mapped to Kafka headers. You can also choose which serializer should be used to fill the payload of the `ProducerRecord`.

For further customization, you can implement your own `KafkaMessageConverter`, and wire it into the `KafkaPublisherConfiguration` and `AsyncFetcher`:

```java
KafkaPublisherConfiguration.<String, byte[]>builder() // the <String, byte[]> defines the type of key and payload, respectively
        .withMessageConverter(customConverter) // the converter needs to match the expected key and payload type
        .build();

AsyncFetcher.builder(configs)
        .withMessageConverter(customConverter)
        .build();
```

## Configuration in Spring Boot

Axon will automatically provide certain Kafka related components based on the availability of beans and/or properties.

To enable a KafkaPublisher, either provide a bean of type `ProducerFactory`, or set `axon.kafka.producer.transaction-id-prefix` in `application.properties` to have auto configuration configure a ProducerFactory with Transactional semantics. In either case, `application.properties` should provide the necessary Kafka Client properties, available under the `axon.kafka` prefix. If none are provided, default settings are used, and `localhost:9092` is used as the bootstrap server.

To enable a `KafkaMessageSource`, either provide a bean of type `ConsumerFactory`, or provide the `axon.kafka.consumer.group-id` setting in `application.properties`. Also make sure all necessary Kafka Client Configuration properties are available under the `axon.kafka` prefix.

Alternatively, you may provide your own `KafkaMessageSource` bean\(s\), in which case Axon will not create the default KafkaMessageSource.