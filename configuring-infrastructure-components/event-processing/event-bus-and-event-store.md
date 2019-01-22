# Event Bus and Event Store

## Event Bus

The `EventBus` is the mechanism that dispatches events to the subscribed event handlers. Axon provides three implementations of the Event Bus: `AxonServerEventStore`, `EmbeddedEventStore` and `SimpleEventBus`. While all three implementations support subscribing and tracking processors \(see [Events Processors](/configuring-infrastructure-components/event-processing/event-processors.md)\), the `AxonServerEventStore` and `EmbeddedEventStore` persists events (see [Event Store](#event-store)), which allows you to replay them at a later stage. The `SimpleEventBus` has a volatile storage and 'forgets' events as soon as they have been published to subscribed components.

`AxonServerEventStore` event bus/store is configured by default.

## Event Store

Event sourcing repositories need an event store to store and load events from aggregates. An event store offers the functionality of an event bus, with the addition that it persists published events, and is able to retrieve events based on an aggregate identifier.

### Axon Server as an event store

Axon provides an event store out of the box, the `AxonServerEventStore`. It connects to the [AxonIQ AxonServer Server](/introduction/axon-server.md) to store and retrieve Events.

{% tabs %}
{% tab title="Axon Configuration API" %}

Declare dependencies:
```
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-server-connector</artifactId>
    <version>${axon.version}</version>
</dependency>
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-configuration</artifactId>
    <version>${axon.version}</version>
</dependency>
```
Configure your application:
```java
// Returns a Configurer instance with default components configured. `AxonServerEventStore` is configured as Event Store by default.
Configurer configurer = DefaultConfigurer.defaultConfiguration();
```
> NOTE: If you exclude `axon-server-connector` dependency you will fallback to 'non-axon-server' event store options: `SimpleEventBus` or `EmbeddedEventStore` (see [below](event-bus-and-event-store.md#embedded-event-store))

{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}

By simply declaring dependency to `axon-spring-boot-starter`, Axon will automatically configure the event bus/event store:
```
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-spring-boot-starter</artifactId>
    <version>${axon.version}</version>
</dependency>

```
{% endtab %}
{% endtabs %}

### Embedded event store

Alternatively, Axon provides non-axon-server option, the `EmbeddedEventStore`. It delegates actual storage and retrieval of events to an `EventStorageEngine`.

There are multiple `EventStorageEngine` implementations available:

#### `JpaEventStorageEngine`

The `JpaEventStorageEngine` stores events in a JPA-compatible data source. The JPA event store stores events in so called entries. These entries contain the serialized form of an event, as well as some fields where metadata is stored for fast lookup of these entries. To use the `JpaEventStorageEngine`, you must have the JPA \(`javax.persistence`\) annotations on your classpath.

By default, the event store needs you to configure your persistence context \(e.g. as defined in `META-INF/persistence.xml` file\) to contain the classes `DomainEventEntry` and `SnapshotEventEntry` \(both in the `org.axonframework.eventsourcing.eventstore.jpa` package\).

Below is an example configuration of a persistence context configuration:

```markup
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="1.0">
    <persistence-unit name="eventStore" transaction-type="RESOURCE_LOCAL"> (1)
        <class>org...eventstore.jpa.DomainEventEntry</class> (2)
        <class>org...eventstore.jpa.SnapshotEventEntry</class>
    </persistence-unit>
</persistence>
```

1. In this sample, there is a specific persistence unit for the event store. You may, however, choose to add the third line to any other persistence unit configuration.
2. This line registers the `DomainEventEntry` \(the class used by the `JpaEventStore`\) with the persistence context.

> **Note**
>
> Axon uses Locking to prevent two threads from accessing the same aggregate. However, if you have multiple JVMs on the same database, this won't help you. In that case, you'd have to rely on the database to detect conflicts. Concurrent access to the event store will result in a Key Constraint Violation, as the table only allows a single Event for an aggregate with any sequence number. Inserting a second event for an existing aggregate with an existing sequence number will result in an error.
>
> The `JpaEventStorageEngine` can detect this error and translate it to a `ConcurrencyException`. However, each database system reports this violation differently. If you register your `DataSource` with the `JpaEventStore`, it will try to detect the type of database and figure out which error codes represent a Key Constraint Violation. Alternatively, you may provide a `PersistenceExceptionTranslator` instance, which can tell if a given exception represents a key constraint violation.
>
> If no `DataSource` or `PersistenceExceptionTranslator` is provided, exceptions from the database driver are thrown as-is.

By default, the `JpaEventStorageEngine` requires an `EntityManagerProvider` implementation that returns the `EntityManager` instance for the `EventStorageEngine` to use. This also allows for application managed persistence contexts to be used. It is the `EntityManagerProvider`'s responsibility to provide a correct instance of the `EntityManager`.

There are a few implementations of the `EntityManagerProvider` available, each for different needs. The `SimpleEntityManagerProvider` simply returns the `EntityManager` instance which is given to it at construction time. This makes the implementation a simple option for container managed contexts. Alternatively, there is the `ContainerManagedEntityManagerProvider`, which returns the default persistence context, and is used by default by the JPA event store.

If you have a persistence unit called `"myPersistenceUnit"` which you wish to use in the `JpaEventStore`, this is what the `EntityManagerProvider` implementation could look like:

```java
public class MyEntityManagerProvider implements EntityManagerProvider {

    private EntityManager entityManager;

    @Override
    public EntityManager getEntityManager() {
        return entityManager;
    }

    @PersistenceContext(unitName = "myPersistenceUnit")
    public void setEntityManager(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
```

By default, the JPA èvent store stores entries in `DomainEventEntry` and `SnapshotEventEntry` entities. While this will suffice in many cases, you might encounter a situation where the meta-data provided by these entities is not enough. Or you might want to store events of different aggregate types in different tables.

If that is the case, you can extend the `JpaEventStorageEngine`. It contains a number of protected methods that you can override to tweak its behavior.

> **Warning**
>
> Note that persistence providers, such as Hibernate, use a first-level cache on their `EntityManager` implementation. Typically, this means that all entities used or returned in queries are attached to the `EntityManager`. They are only cleared when the surrounding transaction is committed or an explicit "clear" is performed inside the transaction. This is especially the case when the queries are executed in the context of a transaction.
>
> To work around this issue, make sure to exclusively query for non-entity objects. You can use JPA's `"SELECT new SomeClass(parameters) FROM ..."` style queries to work around this issue. Alternatively, call `EntityManager.flush()` and `EntityManager.clear()` after fetching a batch of events. Failure to do so might result in `OutOfMemoryException`s when loading large streams of events.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance which has JPA components configured, such as a JPA based Event Store `JpaEventStorageEngine`, a `JpaTokenStore` and `JpaSagaStore`.
Configurer configurer = DefaultConfigurer.jpaConfiguration(entityManagerProvider, transactionManager);
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
// The Event store `EmbeddedEventStore` delegates actual storage and retrieval of events to an `EventStorageEngine`.
@Bean
public EmbeddedEventStore eventStore(EventStorageEngine storageEngine, AxonConfiguration configuration) {
    return EmbeddedEventStore.builder()
            .storageEngine(storageEngine)
            .messageMonitor(configuration.messageMonitor(EventStore.class, "eventStore"))
            .build();
}

// The JpaEventStorageEngine stores events in a JPA-compatible data source
@Bean
public EventStorageEngine storageEngine(Serializer defaultSerializer,
                                        PersistenceExceptionResolver persistenceExceptionResolver,
                                        @Qualifier("eventSerializer") Serializer eventSerializer,
                                        AxonConfiguration configuration,
                                        EntityManagerProvider entityManagerProvider,
                                        TransactionManager transactionManager) {
    return JpaEventStorageEngine.builder()
            .snapshotSerializer(defaultSerializer)
            .upcasterChain(configuration.upcasterChain())
            .persistenceExceptionResolver(persistenceExceptionResolver)
            .eventSerializer(eventSerializer)
            .entityManagerProvider(entityManagerProvider)
            .transactionManager(transactionManager)
            .build();
}
```
> NOTE: If you exclude `axon-server-connector` dependency from `axon-spring-boot-starter` you will have `EmbeddedEventStore` component auto-configured and loaded into Spring Application Context (if concrete implementation of `EventStorageEngine` is available in Spring Application Context). If JPA is detected on the classpath `JpaEventStorageEngine` will be auto-configured as `EventStorageEngine` and you don't need to explicitly define this Spring beans in your configuration.

{% endtab %}
{% endtabs %}

#### JdbcEventStorageEngine

The JDBC event storage engine uses a JDBC Connection to store events in a JDBC compatible data storage. Typically, these are relational databases. Theoretically, anything that has a JDBC driver could be used to back the `JdbcEventStorageEngine`.

Similar to its JPA counterpart, the `JDBCEventStorageEngine` stores events in entries. By default, each event is stored in a single entry, which corresponds with a row in a table. One table is used for events and another for the snapshots.

The `JdbcEventStorageEngine` uses a `ConnectionProvider` to obtain connections. Typically, these connections can be obtained directly from a DataSource. However, Axon will bind these connections to a unit of work, so that a single connection is used in a unit of work. This ensures that a single transaction is used to store all events, even when multiple units of work are nested in the same thread.

> **Note**
>
> Spring users are recommended to use the `SpringDataSourceConnectionProvider` to attach a connection from a `DataSource` to an existing transaction.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. We explicitly set `JdbcEventStorageEngine` as desired engine for Embedded Event Store.
Configurer configurer = DefaultConfigurer.defaultConfiguration()
      .configureEmbeddedEventStore(c -> JdbcEventStorageEngine.builder().connectionProvider(connectionProvider).transactionManager(NoTransactionManager.INSTANCE).build());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
// The Event store `EmbeddedEventStore` delegates actual storage and retrieval of events to an `EventStorageEngine`.
@Bean
public EmbeddedEventStore eventStore(EventStorageEngine storageEngine, AxonConfiguration configuration) {
    return EmbeddedEventStore.builder()
            .storageEngine(storageEngine)
            .messageMonitor(configuration.messageMonitor(EventStore.class, "eventStore"))
            .build();
}

// EventStorageEngine implementation that uses JDBC to store and fetch events.
@Bean
public JdbcEventStorageEngine eventStorageEngine(ConnectionProvider connectionProvider) {
    return JdbcEventStorageEngine.builder()
            .connectionProvider(connectionProvider)
            .transactionManager(NoTransactionManager.INSTANCE)
            .build();

```
> NOTE: If you exclude `axon-server-connector` dependency from `axon-spring-boot-starter` you will have `EmbeddedEventStore` component auto-configured and loaded into Spring Application Context (if concrete implementation of `EventStorageEngine` is available in Spring Application Context). You only need to configure `EventStorageEngine / JdbcEventStorageEngine` Spring bean in your configuration in this case.

{% endtab %}
{% endtabs %}

#### MongoEventStorageEngine

[MongoDB](https://www.mongodb.com/) is a document based NoSQL store. Its scalability characteristics make it suitable for use as an event store. Axon provides the `MongoEventStorageEngine`, which uses MongoDB as backing database. It is contained in the Axon Mongo module \(Maven artifactId `axon-mongo`\).

Events are stored in two separate collections: one for the actual event streams and one for the snapshots.

By default, the `MongoEventStorageEngine` stores each event in a separate document. It is, however, possible to change the `StorageStrategy` used. The alternative provided by Axon is the `DocumentPerCommitStorageStrategy`, which creates a single document for all events that have been stored in a single commit \(i.e. in the same `DomainEventStream`\).

Storing an entire commit in a single document has the advantage that a commit is stored atomically. Furthermore, it requires only a single roundtrip for any number of events. A disadvantage is that it becomes harder to query events directly in the database. When refactoring the domain model, for example, it is harder to "transfer" events from one aggregate to another if they are included in a "commit document".

The MongoDB does not take a lot of configuration. All it needs is a reference to the collections to store the events in, and you're set to go. For production environments, you may want to double check the indexes on your collections.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. We explicitly set `MongoEventStorageEngine` as desired engine for Embedded Event Store.
Configurer configurer = DefaultConfigurer.defaultConfiguration()
      .configureEmbeddedEventStore(c -> MongoEventStorageEngine.builder().mongoTemplate(mongoTemplate).build());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
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
> NOTE: If you exclude `axon-server-connector` dependency from `axon-spring-boot-starter` you will have `EmbeddedEventStore` component auto-configured and loaded into Spring Application Context (if concrete implementation of `EventStorageEngine` is available in Spring Application Context). You only need to configure `EventStorageEngine / MongoEventStorageEngine` Spring bean in your configuration in this case.

{% endtab %}
{% endtabs %}


### Event store utilities

Axon provides a number of Event Storage Engines that may be useful in certain circumstances.

#### In-Memory Event Storage

The `InMemoryEventStorageEngine` keeps the stored events in memory. While it probably outperforms any other event store out there, it is not really meant for long-term production use. However, it is very useful in short-lived tools or tests that require an event store.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. We explicitly set `InMemoryEventStorageEngine` as desired engine for Embedded Event Store.
Configurer configurer = DefaultConfigurer.defaultConfiguration()
      .configureEmbeddedEventStore(c -> new InMemoryEventStorageEngine());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
// The Event store `EmbeddedEventStore` delegates actual storage and retrieval of events to an `EventStorageEngine`.
@Bean
public EmbeddedEventStore eventStore(EventStorageEngine storageEngine, AxonConfiguration configuration) {
    return EmbeddedEventStore.builder()
            .storageEngine(storageEngine)
            .messageMonitor(configuration.messageMonitor(EventStore.class, "eventStore"))
            .build();
}

// The `InMemoryEventStorageEngine` stores each event in memory
@Bean
public EventStorageEngine storageEngine() {
    return new InMemoryEventStorageEngine();
}
```
> NOTE: If you exclude `axon-server-connector` dependency from `axon-spring-boot-starter` you will have `EmbeddedEventStore` component auto-configured and loaded into Spring Application Context (if concrete implementation of `EventStorageEngine` is available in Spring Application Context). You only need to configure `EventStorageEngine / InMemoryEventStorageEngine` Spring bean in your configuration in this case.

{% endtab %}
{% endtabs %}

#### Combining multiple event stores into one

The `SequenceEventStorageEngine` is a wrapper around two other event storage engines. When reading, it returns the events from both event storage engines. Appended events are only appended to the second event storage engine. This is useful in cases where two different implementations of event storage are used for performance reasons, for example. The first would be a larger, but slower event store, while the second is optimized for quick reading and writing.

#### Filtering Stored Events

The `FilteringEventStorageEngine` allows events to be filtered based on a predicate. Only events that match this predicate will be stored. Note that event processors that use the event store as a source of events, may not receive these events, as they are not being stored.

### Influencing the serialization process

Event stores need a way to serialize the event to prepare it for storage. By default, Axon uses the `XStreamSerializer`, which uses [XStream](http://x-stream.github.io/) to serialize events into XML. XStream is reasonably fast and is more flexible than Java Serialization. Furthermore, the result of XStream serialization is human readable. Quite useful for logging and debugging purposes.

The `XStreamSerializer` can be configured. You can define aliases it should use for certain packages, classes or even fields. Besides being a nice way to shorten potentially long names, aliases can also be used when class definitions of events change. For more information about aliases, visit the [XStream website](http://x-stream.github.io/).

Alternatively, Axon also provides the `JacksonSerializer`, which uses [Jackson](https://github.com/FasterXML/jackson) to serialize events into JSON. While it produces a more compact serialized form, it does require that classes stick to the conventions \(or configuration\) required by Jackson.

You may also implement your own serializer, simply by creating a class that implements `Serializer`, and configuring the event store to use that implementation instead of the default.


{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. We explicitly set `JacksonSerializer` as desired event serializer.
Configurer configurer = DefaultConfigurer.defaultConfiguration()
      .configureEventSerializer(c -> JacksonSerializer.builder().build());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```
# somewhere in your `application.properties`

axon.serializer.events=jackson # posible values: java, xstream, jackson
```
or explicitly define your Serializer in Spring context:
```java
// somewhere in your `@Configuration` class
@Qualifier("eventSerializer")
@Bean
public Serializer eventSerializer() {
    return JacksonSerializer.builder().build();
}
```
{% endtab %}
{% endtabs %}


#### Serializing events vs 'the rest'

It is possible to use a different serializer for the storage of events, than all other objects that Axon needs to serializer \(such as commands, snapshots, sagas, etc\). While the `XStreamSerializer`'s capability to serialize virtually anything makes it a very decent default, its output is not always a form that makes it nice to share with other applications. The `JacksonSerializer` creates much nicer output, but requires a certain structure in the objects to serialize. This structure is typically present in events, making it a very suitable event serializer.

If no explicit `eventSerializer` is configured, Events are serialized using the main serializer that has been configured \(which in turn defaults to the `XStreamSerializer`\).
