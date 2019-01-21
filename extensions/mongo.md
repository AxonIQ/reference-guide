# Mongo DB

The `MongoEventStorageEngine` has an `@PostConstruct` annotated method,
 called `ensureIndexes` which will generate the indexes required for correct operation. 
That means, when running in a container that automatically calls `@PostConstruct` handlers,
 the required unique index on "Aggregate Identifier" and "Event Sequence Number" is created when the event store is created.

Note that there is always a balance between query optimization and update speed. 
Load testing is ultimately the best way to discover which indices provide the best performance.

* Normal operational use

  An index is automatically created on `"aggregateIdentifier"`, `"type"` and `"sequenceNumber"` in the domain events 
   \(default name: `"domainevents"`\) collection.
  Additionally, a non-unique index on `"timestamp"` and `"sequenceNumber"` is configured on the domain events 
   \(default name: `"domainevents"`\) collection, for the tracking event processors.
* Snapshotting

  An \(unique\) index on `"aggregateIdentifier"` and `"sequenceNumber"` is automatically created in the snapshot events 
   \(default name: `"snapshotevents"`\) collection.
* Sagas

  Put a \(unique\) index on the `"sagaIdentifier"` in the saga \(default name: `"sagas"`\) collection.
  Put an index on the `"sagaType"`, `"associations.key"` and `"associations.value"` properties in the saga 
   \(default name: `"sagas"`\) collection.

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
