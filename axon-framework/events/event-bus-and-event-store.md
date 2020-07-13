# Event Bus & Event Store

## Event Bus

The `EventBus` is the mechanism that dispatches events to the subscribed event handlers. Axon provides three implementations of the Event Bus: `AxonServerEventStore`, `EmbeddedEventStore` and `SimpleEventBus`. All three implementations support subscribing and tracking processors \(see [Events Processors](event-processors.md)\). However, the `AxonServerEventStore` and `EmbeddedEventStore` persist events \(see [Event Store](event-bus-and-event-store.md)\), which allows you to replay them at a later stage. The `SimpleEventBus` has a volatile storage and 'forgets' events as soon as they have been published to subscribed components.

An `AxonServerEventStore` event bus/store is configured by default.

## Event Store

Event sourcing repositories need an event store to store and load events from aggregates. An event store offers the functionality of an event bus. Additionally, it persists published events and is able to retrieve previous events based on a given aggregate identifier.

### Axon Server as an event store

Axon provides an event store out of the box, the `AxonServerEventStore`. It connects to the[ AxonIQ AxonServer Server ](../../axon-server-introduction.md)to store and retrieve Events.

{% tabs %}
{% tab title="Axon Configuration API" %}
Declare dependencies:

```text
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
// Returns a Configurer instance with default components configured. 
// `AxonServerEventStore` is configured as Event Store by default.
Configurer configurer = DefaultConfigurer.defaultConfiguration();
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
By simply declaring a dependency on `axon-spring-boot-starter`, Axon will automatically configure the event bus/event store:

```text
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-spring-boot-starter</artifactId>
    <version>${axon.version}</version>
</dependency>
```

> **Excluding the Axon Server Connector**
>
> If you exclude the `axon-server-connector` dependency from `axon-spring-boot-starter` the `EmbeddedEventStore` will be auto-configured for you, if a concrete implementation of `EventStorageEngine` is available. If JPA or JDBC is detected on the classpath, a `JpaEventStorageEngine` or `JdbcEventStorageEngine` will respectively be auto-configured as the `EventStorageEngine`. In absence of either, the auto configuration falls back to the `SimpleEventBus`.
{% endtab %}
{% endtabs %}

### Embedded event store

Alternatively, Axon provides a non-axon-server option, the `EmbeddedEventStore`. It delegates the actual storage and retrieval of events to an `EventStorageEngine`.

There are multiple `EventStorageEngine` implementations available:

#### `JpaEventStorageEngine`

The `JpaEventStorageEngine` stores events in a JPA-compatible data source. The JPA event store stores events in entries. These entries contain the serialized form of an event, as well as some fields where metadata is stored for fast lookup of these entries. To use the `JpaEventStorageEngine`, you must have the JPA \(`javax.persistence`\) annotations on your classpath.

By default, the event store needs you to configure your persistence context \(e.g. as defined in the `META-INF/persistence.xml` file\) to contain the classes `DomainEventEntry` and `SnapshotEventEntry` \(both of these classes are located in the `org.axonframework.eventsourcing.eventstore.jpa` package\).

Below is an example configuration of a persistence context configuration:

```markup
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="1.0">
    <persistence-unit name="eventStore" transaction-type="RESOURCE_LOCAL"> (1)
        <class>org...eventstore.jpa.DomainEventEntry</class> (2)
        <class>org...eventstore.jpa.SnapshotEventEntry</class>
    </persistence-unit>
</persistence>
```

1. In this example, there is a specific persistence unit for the event store.

   You may, however, choose to add the third line to any other persistence unit configuration.

2. This line registers the `DomainEventEntry` \(the class used by the `JpaEventStore`\) with the persistence context.

> **Unique Key Constraint Consideration**
>
> Axon uses locking to prevent two threads from accessing the same aggregate. However, if you have multiple JVMs using the same database, this won't help you. In that case, you'd have to rely on the database to detect conflicts. Concurrent access to the event store will result in a Key Constraint Violation, as the table only allows a single event for a given aggregate and sequence number. Therefore, inserting a second event for an existing aggregate with an existing sequence number will result in an error.
>
> The `JpaEventStorageEngine` can detect this error and translate it to a `ConcurrencyException`. However, each database system reports this violation differently. If you register your `DataSource` with the `JpaEventStore`, it will try to detect the type of database and figure out which error codes represent a Key Constraint Violation. Alternatively, you may provide a `PersistenceExceptionTranslator` instance, which can tell if a given exception represents a key constraint violation.
>
> If no `DataSource` or `PersistenceExceptionTranslator` is provided, exceptions from the database driver are thrown as-is.

By default, the `JpaEventStorageEngine` requires an `EntityManagerProvider` implementation that returns the `EntityManager` instance for the `EventStorageEngine` to use. This also allows application managed persistence contexts to be used. It is the `EntityManagerProvider`'s responsibility to provide a correct instance of the `EntityManager`.

There are a few implementations of the `EntityManagerProvider` available, each for different needs. The `SimpleEntityManagerProvider` simply returns the `EntityManager` instance which is given to it at construction time. This makes the implementation a simple option for container managed contexts. Alternatively, there is the `ContainerManagedEntityManagerProvider`, which returns the default persistence context, and is used by default by the JPA event store.

If you have a persistence unit called `"myPersistenceUnit"` which you wish to use in the `JpaEventStore`, the `EntityManagerProvider` implementation could look like this:

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

By default, the JPA event store stores entries in `DomainEventEntry` and `SnapshotEventEntry` entities. While this will suffice in many cases, you might encounter a situation where the metadata provided by these entities is not enough. It is also possible that you might want to store events for different aggregate types in different tables.

If that is the case, you can extend the `JpaEventStorageEngine`. It contains a number of protected methods that you can override to tweak its behavior.

> **Warning**
>
> Note that persistence providers, such as Hibernate, use a first-level cache in their `EntityManager` implementation. Typically, this means that all entities used or returned in queries are attached to the `EntityManager`. They are only cleared when the surrounding transaction is committed or an explicit "clear" is performed inside the transaction. This is especially the case when the queries are executed in the context of a transaction.
>
> To work around this issue, make sure to exclusively query for non-entity objects. You can use JPA's `"SELECT new SomeClass(parameters) FROM ..."` style queries to work around this issue. Alternatively, call `EntityManager.flush()` and `EntityManager.clear()` after fetching a batch of events. Failure to do so might result in `OutOfMemoryException`s when loading large streams of events.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance which has JPA components configured,
//  such as a JPA based Event Store `JpaEventStorageEngine`, a `JpaTokenStore` and `JpaSagaStore`.
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

> **Excluding the Axon Server Connector**
>
> If you exclude the `axon-server-connector` dependency from `axon-spring-boot-starter` the `EmbeddedEventStore` will be auto-configured for you, if a concrete implementation of `EventStorageEngine` is available. If JPA or JDBC is detected on the classpath, a `JpaEventStorageEngine` or `JdbcEventStorageEngine` will respectively be auto-configured as the `EventStorageEngine`. In absence of either, the auto configuration falls back to the `SimpleEventBus`.
{% endtab %}
{% endtabs %}

#### JdbcEventStorageEngine

The JDBC event storage engine uses a JDBC Connection to store events in a JDBC compatible data storage. Typically, these are relational databases. Theoretically, anything that has a JDBC driver could be used to back the `JdbcEventStorageEngine`.

Similar to its JPA counterpart, the `JDBCEventStorageEngine` stores events in entries. By default, each event is stored in a single entry, which corresponds with a row in a table. One table is used for events and another for snapshots.

The `JdbcEventStorageEngine` uses a `ConnectionProvider` to obtain connections. Typically, these connections can be obtained directly from a `DataSource`. However, Axon will bind these connections to a unit of work, so that a single connection is used within a unit of work. This ensures that a single transaction is used to store all events, even when multiple units of work are nested in the same thread.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. 
// We explicitly set `JdbcEventStorageEngine` as desired engine for Embedded Event Store.
Configurer configurer = DefaultConfigurer.defaultConfiguration()
    .configureEmbeddedEventStore(
        c -> JdbcEventStorageEngine.builder()
            .connectionProvider(connectionProvider)
            .transactionManager(NoTransactionManager.INSTANCE)
            .build()
    );
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
```

> **Data sources providers with Spring**
>
> Spring users are recommended to use the `SpringDataSourceConnectionProvider` to attach a connection from a `DataSource` to an existing transaction.

```java
// EventStorageEngine implementation that uses JDBC to store and fetch events.
@Bean
public JdbcEventStorageEngine eventStorageEngine(ConnectionProvider connectionProvider) {
    return JdbcEventStorageEngine.builder()
            .connectionProvider(connectionProvider)
            .transactionManager(NoTransactionManager.INSTANCE)
            .build();
```

> **Excluding the Axon Server Connector**
>
> If you exclude the `axon-server-connector` dependency from `axon-spring-boot-starter` the `EmbeddedEventStore` will be auto-configured for you, if a concrete implementation of `EventStorageEngine` is available. When it is desired to use JDBC as the Event Storage approach, you can provide a `JdbcEventStorageEngine` bean directly or let it be auto configured by Axon by exposing a `DataSource` bean.
{% endtab %}
{% endtabs %}

> **SQL Statement Customizability**
>
> Databases have slight deviations between what's the optimal SQL statement to perform in differing scenarios. As optimizing for all possibilities out there is beyond the scope of the framework, it is possible to adjust the default statements being used.
>
> Check the `JdbcEventStorageEngineStatements` utility class for the default statements used by the `JdbcEventStorageEngine`. Further more, the `org.axonframework.eventsourcing.eventstore.jdbc.statements` package contains the set of adjustable statements. Each of these statement-builders can be customized through the `JdbcEventStorageEngine.Builder`.

#### MongoEventStorageEngine

[MongoDB](https://www.mongodb.com/) is a document based NoSQL store. Its scalability characteristics make it suitable for use as an event store. Axon provides the `MongoEventStorageEngine`, which uses MongoDB as a backing database. It is contained in the Axon Mongo module \(Maven artifactId `axon-mongo`\).

Events are stored in two separate collections: one for the event streams and one for snapshots.

By default, the `MongoEventStorageEngine` stores each event in a separate document. It is, however, possible to change the `StorageStrategy` used. The alternative provided by Axon is the `DocumentPerCommitStorageStrategy`, which creates a single document for all events that have been stored in a single commit \(i.e. in the same `DomainEventStream`\).

The advantage of storing an entire commit in a single document is that commit is stored atomically. Furthermore, it requires only a single roundtrip for any number of events. The disadvantage is that it becomes harder to query events directly in the database. For example, when refactoring the domain model it is harder to "transfer" events from one aggregate to another if they are included in a "commit document".

The `MongoEventStorageEngine` does not require a lot of configuration. All it needs is a reference to the collections to store the events in, and you're set to go. For production environments, you may want to double check the indexes on your collections.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. 
// We explicitly set `MongoEventStorageEngine` as desired engine for Embedded Event Store.
Configurer configurer = DefaultConfigurer.defaultConfiguration()
    .configureEmbeddedEventStore(
        c -> MongoEventStorageEngine.builder().mongoTemplate(mongoTemplate).build()
    );
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

> **Excluding the Axon Server Connector**
>
> If you exclude the `axon-server-connector` dependency from `axon-spring-boot-starter` the `EmbeddedEventStore` will be auto-configured for you, if a concrete implementation of `EventStorageEngine` is available. When it is desired to use Mongo as the Event Storage approach, this means providing a `MongoEventStorageEngine` bean.
{% endtab %}
{% endtabs %}

### Event store utilities

Axon provides a number of Event Storage Engines that may be useful in certain circumstances.

#### In-Memory Event Storage

The `InMemoryEventStorageEngine` keeps stored events in memory. While it probably outperforms any other event store out there, it is not really meant for long-term production use. However, it is very useful in short-lived tools or tests that require an event store.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. 
// We explicitly set `InMemoryEventStorageEngine` as desired engine for Embedded Event Store.
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

> **Excluding the Axon Server Connector**
>
> If you exclude the `axon-server-connector` dependency from `axon-spring-boot-starter` the `EmbeddedEventStore` will be auto-configured for you, if a concrete implementation of `EventStorageEngine` is available. When it is desired to use an in-memory Event Storage approach, this means providing an `InMemoryEventStorageEngine` Bean.
{% endtab %}
{% endtabs %}

#### Combining multiple event stores into one

The `SequenceEventStorageEngine` is a wrapper around two other event storage engines. When reading, it returns the events from both event storage engines. Appended events are only appended to the second event storage engine. This is useful in cases where two different implementations of event storage are used for performance reasons, for example. The first would be a larger, but slower event store, while the second is optimized for quick reading and writing.

#### Filtering Stored Events

The `FilteringEventStorageEngine` allows events to be filtered based on a predicate. Only events that match the given predicate will be stored. Note that event processors that use the event store as a source of events may not receive these events because they are not being stored.

### Influencing the serialization process

Event stores need a way to serialize the event to prepare it for storage. By default, Axon uses the `XStreamSerializer`, which uses [XStream](http://x-stream.github.io/) to serialize events into XML. XStream is reasonably fast and is more flexible than Java Serialization. Furthermore, the result of XStream serialization is human readable. This makes it quite useful for logging and debugging purposes.

The `XStreamSerializer` can be configured. You can define aliases it should use for certain packages, classes or even fields. Besides being a nice way to shorten potentially long names, aliases can also be used when class definitions of events change. For more information about aliases, visit the [XStream website](http://x-stream.github.io/).

Alternatively, Axon also provides the `JacksonSerializer`, which uses [Jackson](https://github.com/FasterXML/jackson) to serialize events into JSON. While it produces a more compact serialized form, it does require that classes stick to the conventions \(or configuration\) required by Jackson.

You may also implement your own serializer, simply by creating a class that implements `Serializer`, and configuring the event store to use that implementation instead of the default.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
// Returns a Configurer instance with default components configured. 
// We explicitly set `JacksonSerializer` as desired event serializer.
Configurer configurer = DefaultConfigurer.defaultConfiguration()
      .configureEventSerializer(c -> JacksonSerializer.builder().build());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
You can specify a serializer in your `application.properties`:

```text
# somewhere in your `application.properties`

axon.serializer.events=jackson # posible values: java, xstream, jackson
```

Alternatively, you can explicitly define your Serializer in the Spring context:

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

It is possible to use a different serializer for the storage of events, than all other objects that Axon needs to serialize \(such as commands, snapshots, sagas, etc\). While the `XStreamSerializer`'s capability to serialize virtually anything makes it a very decent default, its output is not always a form that makes it nice to share with other applications. The `JacksonSerializer` creates much nicer output, but requires a certain structure in the objects to serialize. This structure is typically present in events, making it a very suitable event serializer.

If no explicit `eventSerializer` is configured, events are serialized using the main serializer that has been configured \(which defaults to the `XStreamSerializer`\).

