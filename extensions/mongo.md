# Mongo

The `MongoEventStorageEngine` has an `@PostConstruct` annotated method, called `ensureIndexes` which will generate the indexes required for correct operation. That means, when running in a container that automatically calls `@PostConstruct` handlers, the required unique index on "Aggregate Identifier" and "Event Sequence Number" is created when the event store is created.

Note that there is always a balance between query optimization and update speed. Load testing is ultimately the best way to discover which indices provide the best performance.

* Normal operational use

  An index is automatically created on `"aggregateIdentifier"`, `"type"` and `"sequenceNumber"` in the domain events \(default name: `"domainevents"`\) collection. Additionally, a non-unique index on `"timestamp"` and `"sequenceNumber"` is configured on the domain events \(default name: `"domainevents"`\) collection, for tracking event processors.

* Snapshotting

  A \(unique\) index on `"aggregateIdentifier"` and `"sequenceNumber"` is automatically created in the snapshot events \(default name: `"snapshotevents"`\) collection.

* Sagas

  Put a \(unique\) index on the `"sagaIdentifier"` in the saga \(default name: `"sagas"`\) collection. Put an index on the `"sagaType"`, `"associations.key"` and `"associations.value"` properties in the saga \(default name: `"sagas"`\) collection.

{% hint style="info" %}
In pre Axon Framework 3 release we found MongoDb to be a very good fit as an Event Store. However with the introduction of Tracking Event Processors and how they track their events, we have encountered some inefficiencies in regards to the Mongo Event Store implementation. We recommend using a built-for-purpose event store like [Axon Server](../axon-server-introduction.md), or alternatively an RDBMS based \(the JPA or JDBC implementations for example\), and would only suggest to use Mongo for this use case if you have found its performance to be beneficial for your application.
{% endhint %}

## Configuration in Spring Boot

```java
// The Event store `EmbeddedEventStore` delegates actual storage and retrieval of events to an `EventStorageEngine`.
@Bean
public EmbeddedEventStore eventStore(EventStorageEngine storageEngine, AxonConfiguration configuration) {
    return EmbeddedEventStore.builder()
            .storageEngine(storageEngine)
            .messageMonitor(configuration.messageMonitor(EventStore.class, "eventStore"))
            .build();
}

// The `MongoEventStorageEngine` stores each event in a separate MongoDB document
@Bean
public EventStorageEngine storageEngine(MongoClient client) {
    return MongoEventStorageEngine.builder().mongoTemplate(DefaultMongoTemplate.builder().mongoDatabase(client).build()).build();
}
```

