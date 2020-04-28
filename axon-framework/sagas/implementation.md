# Implementation

A Saga is a special type of event listener: one that manages a business transaction. Some transactions could be running for days or even weeks, while others are completed within a few milliseconds. In Axon, each instance of a Saga is responsible for managing a single business transaction. That means a Saga maintains state necessary to manage that transaction, continuing it or taking compensating actions to roll back any actions already taken. Typically, and contrary to regular event listeners, a saga has a starting point and an end, both triggered by events. While the starting point of a saga is usually very clear, there could be many ways for a saga to end.

In Axon, sagas are classes that define one or more `@SagaEventHandler` methods. Unlike regular event handlers, multiple instances of a saga may exist at any time. Sagas are managed by a single event processor \(Tracking or Subscribing\), which is dedicated to dealing with events for that specific saga type.

## Life cycle

A single Saga instance is responsible for managing a single transaction. That means you need to be able to indicate the start and end of a saga's life cycle.

In a saga, event handlers are annotated with `@SagaEventHandler`. If a specific event signifies the start of a transaction, add another annotation to that same method: `@StartSaga`. This annotation will create a new saga and invoke its event handler method when a matching event is published.

By default, a new saga is only started if no suitable existing saga \(of the same type\) can be found. You can also force the creation of a new saga instance by setting the `forceNew` property on the `@StartSaga` annotation to `true`.

Ending a saga can be done in two ways. If a certain event always indicates the end of a saga its life cycle, annotate that event handler on the saga with `@EndSaga`. The saga its life cycle will be ended after the invocation of the handler. Alternatively, you can call `SagaLifecycle.end()` from inside the saga to end the life cycle. This allows you to conditionally end the saga.

## Event handling

Event handling in a saga is quite comparable to that of a regular event listener. The same rules for method and parameter resolution are valid here. There is one major difference, though. While there is a single instance of an event listener that deals with all incoming events, multiple instances of a saga may exist, each interested in different events. For example, a saga that manages a transaction around an Order with Id "1" will not be interested in events regarding Order "2", and vice versa.

Instead of publishing all events to all saga instances \(which would be a complete waste of resources\), Axon will only publish events containing properties that the saga has been associated with. This is done using `AssociationValue`s. An `AssociationValue` consists of a key and a value. The key represents the type of identifier used, for example "orderId" or "order". The value represents the corresponding value, "1" or "2" in the previous example.

The order in which `@SagaEventHandler` annotated methods are evaluated is identical to that of `@EventHandler` methods \(see [Annotated event handler](../events/event-handlers.md)\). A method matches if the parameters of the handler method match the incoming event, and if the saga has an association with the property defined on the handler method.

The `@SagaEventHandler` annotation has two attributes, of which `associationProperty` is the most important one. This is the name of the property on the incoming event that should be used to find associated sagas. The key of the association value is the name of the property. The value is the value returned by property its getter method.

For example, consider an incoming Event with a method `String getOrderId()`, which returns "123". If a method accepting this event is annotated with `@SagaEventHandler(associationProperty="orderId")`, this Event is routed to all Sagas that have been associated with an `AssociationValue` with key "orderId" and value "123". This may either be exactly one, more than one, or even none at all.

Sometimes, the name of the property you want to associate with is not the name of the association you want to use. For example, you have a Saga that matches "Sell orders" against "Buy orders", you could have a transaction object that contains the "buyOrderId" and a "sellOrderId". If you want the saga to associate the "sellOrderId" value as "orderId", you can define a different `keyName` in the `@SagaEventHandler` annotation. It would then become `@SagaEventHandler(associationProperty="sellOrderId", keyName="orderId")`

## Injecting Resources

Sagas generally do more than just maintaining state based on events. They interact with external components. To do so, they need access to the resources necessary to address to components. Usually, these resources aren't really part of the saga and its state and these resources should not be persisted as such. However, once a saga is reconstructed, these resources must be injected before an event is routed to that instance.

For that purpose, there is the `ResourceInjector`. It is used by the `SagaRepository` to inject resources into a saga. Axon provides a `SpringResourceInjector`, which injects annotated fields and methods with resources from the Application Context. Axon also provides a `SimpleResourceInjector`, which injects resources that have been registered with it into `@Inject` annotated methods and fields.

> **Tip**
>
> Since resources should not be persisted with the saga, make sure to add the `transient` keyword to those fields. This will prevent the serialization mechanism to attempt to write the contents of these fields to the repository. The repository will automatically re-inject the required resources after a saga has been deserialized.

The `SimpleResourceInjector` allows for a pre-specified collection of resources to be injected. It scans the \(setter\) methods and fields of a Saga to find ones that are annotated with `@Inject`.

When using the Configuration API, Axon will default to the `ConfigurationResourceInjector`. It will inject any resource available in the configuration. Components like the `EventBus`, `EventStore`, `CommandBus` and `CommandGateway` are available by default. You can also register your own components using `configurer.registerComponent()`.

The `SpringResourceInjector` uses Spring's dependency injection mechanism to inject resources into a Saga. This means you can use setter injection or direct field injection if you require. The method or field to be injected needs to be annotated in order for Spring to recognize it as a dependency, for example with `@Autowired`.

## Saga Infrastructure

Events need to be redirected to the appropriate saga instances. To do so, some infrastructure classes are required. The most important components are the `SagaManager` and the `SagaRepository`.

### Saga Manager

Like any component that handles events, the processing is done by an event processor. However, sagas are not singleton instances handling events. They have individual life cycles which need to be managed.

Axon supports life cycle management through the `AnnotatedSagaManager`, which is provided to an event processor to perform the actual invocation of handlers. It is initialized using the type of the saga to manage, as well as a `SagaRepository` where sagas of that type can be stored and retrieved. A single `AnnotatedSagaManager` can only manage a single saga type.

When using the Configuration API, Axon will use sensible defaults for most components. However, it is highly recommended to define a `SagaStore` implementation to use. The `SagaStore` is the mechanism that 'physically' stores the saga instances somewhere. The `AnnotatedSagaRepository` \(the default\) uses the `SagaStore` to store and retrieve Saga instances as they are required.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
        configurer.eventProcessing(eventProcessingConfigurer -> eventProcessingConfigurer
                .registerSaga(MySaga.class,
                              // Axon defaults to an in-memory SagaStore,
                              // defining another is recommended
                              sagaConfigurer -> sagaConfigurer.configureSagaStore(c -> new JpaSagaStore(...))));

// alternatively, it is possible to register a single SagaStore for all Saga types:
configurer.registerComponent(SagaStore.class, c -> new JpaSagaStore(...));
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Saga(sagaStore = "mySagaStore")
public class MySaga {...}
...
// somewhere in configuration
@Bean
public SagaStore mySagaStore() {
    return new MongoSagaStore(...); // default is JpaSagaStore
}
```
{% endtab %}
{% endtabs %}

### Saga repository and saga store

The `SagaRepository` is responsible for storing and retrieving sagas, for use by the `SagaManager`. It is capable of retrieving specific saga instances by their identifier as well as by their association values.

There are some special requirements, however. Since concurrency handling in sagas is a very delicate procedure, the repository must ensure that for each conceptual saga instance \(with an equal identifier\) only a single instance exists in the JVM.

Axon provides the `AnnotatedSagaRepository` implementation, which allows the lookup of saga instances while guaranteeing that only a single instance of the saga may be accessed at the same time. It uses a `SagaStore` to perform the actual persistence of saga instances.

The choice for the implementation to use depends mainly on the storage engine used by the application. Axon provides the `JdbcSagaStore`, `InMemorySagaStore`, `JpaSagaStore` and `MongoSagaStore`.

In some cases, applications benefit from caching saga instances. In that case, there is a `CachingSagaStore` which wraps another implementation to add caching behavior. Note that the `CachingSagaStore` is a write-through cache, which means save operations are always immediately forwarded to the backing Store, to ensure data safety.

#### JpaSagaStore

The `JpaSagaStore` uses JPA to store the state and association values of sagas. Sagas themselves do not need any JPA annotations; Axon will serialize the sagas using a `Serializer` \(similar to event serialization, you can choose between an `XStreamSerializer`, `JacksonSerializer` or `JavaSerializer`, which can be set by configuring the default `Serializer` in your application. For more details, see [Serializers](../events/event-serialization.md).

The `JpaSagaStore` is configured with an `EntityManagerProvider`, which provides access to an `EntityManager` instance to use. This abstraction allows for the use of both application managed and container managed `EntityManager`s. Optionally, you can define the serializer to serialize the Saga instances with. Axon defaults to the `XStreamSerializer`.

#### JdbcSagaStore

The `JdbcSagaStore` uses plain JDBC to store stage instances and their association values. Similar to the `JpaSagaStore`, saga instances do not need to be aware of how they are stored. They are serialized using a serializer.

The `JdbcSagaStore` is initialized with either a `DataSource` or a `ConnectionProvider`. While not required, when initializing with a `ConnectionProvider`, it is recommended to wrap the implementation in a `UnitOfWorkAwareConnectionProviderWrapper`. It will check the current Unit of Work for an already open database connection, to ensure that all activity within a unit of work is done on a single connection.

Unlike JPA, the `JdbcSagaRepository` uses plain SQL statements to store and retrieve information. This may mean that some operations depend on the database specific SQL dialect. It may also be the case that certain database vendors provide non-standard features that you would like to use. To allow for this, you can provide your own `SagaSqlSchema`. The `SagaSqlSchema` is an interface that defines all the operations the repository needs to perform on the underlying database. It allows you to customize the SQL statement executed for each operation. The default is the `GenericSagaSqlSchema`. Other implementations available are `PostgresSagaSqlSchema`, `Oracle11SagaSqlSchema` and `HsqlSagaSchema`.

#### MongoSagaStore

The `MongoSagaStore` stores the saga instances and their associations in a MongoDB database. The `MongoSagaStore` stores all sagas in a single collection in a MongoDB database. For each saga instance, a single document is created.

The `MongoSagaStore` also ensures that at any time, only a single Saga instance exists for any unique Saga in a single JVM. This ensures that no state changes are lost due to concurrency issues.

The `MongoSagaStore` is initialized using a `MongoTemplate` and optionally a `Serializer`. The `MongoTemplate` provides a reference to the collection to store the sagas in. Axon provides the `DefaultMongoTemplate`, which takes a `MongoClient` instance as well as the database name and name of the collection to store the sagas in. The database name and collection name may be omitted. In that case, they default to `"axonframework"` and `"sagas"`, respectively.

### Caching

If a database backed saga storage is used, saving and loading saga instances may be a relatively expensive operation. In situations where the same saga instance is invoked multiple times within a short time span, a cache can be especially beneficial to the application's performance.

Axon provides the `CachingSagaStore` implementation. It is a `SagaStore` that wraps another one, which does the actual storage. When loading sagas or association values, the `CachingSagaStore` will first consult its caches, before delegating to the wrapped repository. When storing information, all calls are always delegated to ensure that the backing storage always has a consistent view on the saga's state.

To configure caching, simply wrap any `SagaStore` in a `CachingSagaStore`. The constructor of the `CachingSagaStore` takes three parameters: 1. The `SagaStore` to wrap 2. The cache to use for association values 3. The cache to use for saga instances

The latter two arguments may refer to the same cache, or to different ones. This depends on the eviction requirements of your specific application.

