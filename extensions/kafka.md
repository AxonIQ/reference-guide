# Apache Kafka

{% hint style="info" %}

The Kafka Extension currently only has a Release Candidate. 
Due to this, minor releases of the extension or Axon Framework may include breaking changes to the APIs.

{% endhint %}

Apache Kafka is a very popular system for publishing and consuming events. 
It's architecture is fundamentally different from most messaging systems, and combines speed with reliability.

Axon provides an extension dedicated to _publishing_ and _receiving_ event messages from Kafka.
The Kafka Extensions should be regarded as an alternative approach to distributing events,
 besides (the default) Axon Server.

The implementation of the extension can be found in [this](https://github.com/AxonFramework/extension-kafka) repository,
 which also contains a [sample project](https://github.com/AxonFramework/extension-kafka/tree/master/kafka-axon-example) using the extension.

To use the Kafka Extension components from Axon, make sure the `axon-kafka` module is available on the classpath.
Using the extension requires setting up and configuring Kafka following your project's requirements.
How this is achieved is outside of the scope of this reference guide
 and should be found in Kafka's [documentation](https://kafka.apache.org/).

{% hint style="info" %}

Note that Kafka is a perfectly fine event distribution mechanism, but it is not a good fit for an event store.
Along those lines this extension **only** provides the means to distributed Axon's events through Kafka.
Due to this the extension cannot be used to event source aggregates, as this requires an event store implementation.
Therefor we recommend using a built-for-purpose event store like [Axon Server](../introduction/axon-server.md),
 or alternatively an RDBMS based \(the JPA or JDBC implementations for example\).

{% endhint %}

## Publishing Events to Kafka

When Event Messages are published to an Event Bus \(or Event Store\),
 they can be forwarded to a Kafka topic using the `KafkaPublisher`. 
To achieve this it will utilize a Kafka `Producer`, retrieved through Axon's `ProducerFactory`.
The `KafkaPublisher` in turn receives the events to publish from a `KafkaEventPublisher`.

Since the `KafkaEventPublisher` is an event message handler in Axon terms, we can provide it to any
 [Event Processor](../configuring-infrastructure-components/event-processing/event-processors.md) to receive the
 published events.
The choice of event processor which brings differing characteristics for event publication to Kafka:

* **Subscribing Event Processor** - publication of the messages to Kafka will happen in the same thread 
\(and Unit of Work\) that published the events to the event bus. This approach ensures failure to send to Kafka enforces
 failure of the initial event publication on the event bus
* **Tracking Event Processor** - publication of the messages to Kafka is run in a different thread \(and Unit of Work\)
 then that which published the events to the event bus. 
 This approach ensure the event has been published on the event bus regardless of whether publication to Kafka works

When settings up event publication it is also important to take into account which `ConfirmationMode` is used.
The `ConfirumationMode` influences the process of actually producing an event message on a Kafka topic,
 but also what kind of `Producer` the `ProducerFactory` will instantiate:

* **Transactional** - This will require the `Producer` to start, commit and (in case of failure) rollback the
 transaction of publishing an event message. 
 Alongside this, it will create a pool of `Producer` instances in the `ProducerFactory` to avoid continuous creation of
  new ones, requiring the user to provide a "transactional id prefix" to uniquely identify every `Producer` in the pool.
* **Wait-for-Ack** - Setting "WAIT_FOR_ACK" as the `ConfirmationMode` will require the `Producer` instance to wait for
 a default of 1 second (configurable on the `KafkaPublisher`) until the event message publication has ben acknowledged.
 Alongside this, it will create a single, shareable `Producer` instance from within the `ProducerFactory`.  
* **None** - This is the _default_ mode, which only ensures a single,
 shareable `Producer` instance from within the `ProducerFactory`.

### Configuring Event Publication to Kafka

The `KafkaPublisher` takes a `KafkaPublisherConfiguration` instance,
 which provides the different values and settings required to publish events to Kafka.

```java
KafkaPublisherConfiguration configuration = KafkaPublisherConfiguration.<String, byte[]>builder()
        .withProducerFactory(factory) // the factory for creating the actual client instances for sending events to kafka
        .withTopic("topic") // the topic to send the events to. Defaults to 'Axon.Events'
        .build();

KafkaPublisher<String, byte[]> publisher = new KafkaPublisher<>(configuration); // create the publisher itself

publisher.start(); // to start publishing all events
```

Axon provides a `DefaultProducerFactory`, which attempts to reuse created instances to avoid continuous creation of new ones. 
It's creation uses a similar builder pattern. 
The builder requires a `configs` Map, which are the settings to use for the Kafka client, such as the Kafka instance locations. 
Please check the Kafka guide for the possible settings and their values.

```java
DefaultProducerFactory.builder(configs)
        .withConfirmationMode(ConfirmationMode.WAIT_FOR_ACK) // either TRANSACTIONAL, WAIT_FOR_ACK or NONE (default)
        .build();

// or, to create a transactional ProducerFactory
DefaultProducerFactory.builder(configs)
        .withTransactionalIdPrefix("myTxPrefix") // this will also set ConfirmationMode to TRANSACTIONAL
        .build();
```

Note that the `DefaultProducerFactory` needs to be `shutDown` properly,
 to ensure all producer instances are properly closed.

## Consuming Events from Kafka

Messages can be consumed by Tracking Event Processors by configuring a `KafkaMessageSource`. 
This message source uses a `Fetcher` to retrieve the actual messages from Kafka. 
You can either use the `AsyncFetcher`, or provide your own.

The `AsyncFetcher` is initialized using a builder, which requires the Kafka Configuration to initialize the client. 
Please check the Kafka guide for the possible settings and their values.

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

The `AsyncFetcher` doesn't need to be explicitly started, as it will start when the first processors connect to it. 
It does need to be shut down, to ensure any thread pool or active connections are properly closed.

## Customizing message format

By default, Axon uses the `DefaultKafkaMessageConverter` to convert an `EventMessage` to a Kafka `ProducerRecord`
 and an `ConsumerRecord` back into an `EventMessage`. 
This implementation already allows for some customization,
 such as how the `Message`'s `MetaData` is mapped to Kafka headers. 
You can also choose which serializer should be used to fill the payload of the `ProducerRecord`.

For further customization, you can implement your own `KafkaMessageConverter`,
 and wire it into the `KafkaPublisherConfiguration` and `AsyncFetcher`:

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

To enable a KafkaPublisher, either provide a bean of type `ProducerFactory`,
 or set `axon.kafka.producer.transaction-id-prefix` in `application.properties` to have auto configuration configure a ProducerFactory with Transactional semantics. 
In either case, `application.properties` should provide the necessary Kafka Client properties,
 available under the `axon.kafka` prefix. 
If none are provided, default settings are used, and `localhost:9092` is used as the bootstrap server.

To enable a `KafkaMessageSource`, either provide a bean of type `ConsumerFactory`,
 or provide the `axon.kafka.consumer.group-id` setting in `application.properties`. 
Also make sure all necessary Kafka Client Configuration properties are available under the `axon.kafka` prefix.

Alternatively, you may provide your own `KafkaMessageSource` bean\(s\),
 in which case Axon will not create the default KafkaMessageSource.