# Sagas

## Managing complex business transactions

Not every command is able to completely execute in a single ACID transaction. A very common example that pops up quite often as an argument for transactions is the money transfer. It is often believed that an atomic and consistent transaction is absolutely required to transfer money from one account to another. Well, it's not. On the contrary, it is quite impossible to do. What if money is transferred from an account on Bank A, to another account on Bank B? Does Bank A acquire a lock in Bank B's database? If the transfer is in progress, is it strange that Bank A has deducted the amount, but Bank B hasn't deposited it yet? Not really, it's "underway". On the other hand, if something goes wrong while depositing the money on Bank B's account, Bank A's customer would want his money back. So we do expect some form of consistency, ultimately.

While ACID transactions are not necessary or even impossible in some cases, some form of transaction management is still required. Typically, these transactions are referred to as BASE transactions: **B**asic **A**vailability, **S**oft state, **E**ventual consistency. Contrary to ACID, BASE transactions cannot be easily rolled back. To roll back, compensating actions need to be taken to revert anything that has occurred as part of the transaction. In the money transfer example, a failure at Bank B to deposit the money, will refund the money in Bank A.

In CQRS, Sagas can be used to manage these BASE transactions. They respond on Events and may dispatch Commands, invoke external applications, etc. In the context of Domain Driven Design, it is not uncommon for Sagas to be used as coordination mechanism between several bounded contexts.

## Saga

A Saga is a special type of Event Listener: one that manages a business transaction. Some transactions could be running for days or even weeks, while others are completed within a few milliseconds. In Axon, each instance of a Saga is responsible for managing a single business transaction. That means a Saga maintains state necessary to manage that transaction, continuing it or taking compensating actions to roll back any actions already taken. Typically, and contrary to regular Event Listeners, a Saga has a starting point and an end, both triggered by Events. While the starting point of a Saga is usually very clear, there could be many ways for a Saga to end.

In Axon, Sagas are classes that define one or more `@SagaEventHandler` methods. Unlike regular Event Handlers, multiple instances of a Saga may exist at any time. Sagas are managed by a single Processor \(Tracking or Subscribing\), which is dedicated to dealing with Events for that specific Saga type.

### Life Cycle

A single Saga instance is responsible for managing a single transaction. That means you need to be able to indicate the start and end of a Saga's Life Cycle.

In a Saga, Event Handlers are annotated with `@SagaEventHandler`. If a specific Event signifies the start of a transaction, add another annotation to that same method: `@StartSaga`. This annotation will create a new saga and invoke its event handler method when a matching Event is published.

By default, a new Saga is only started if no suitable existing Saga \(of the same type\) can be found. You can also force the creation of a new Saga instance by setting the `forceNew` property on the `@StartSaga` annotation to `true`.

Ending a Saga can be done in two ways. If a certain Event always indicates the end of a Saga's life cycle, annotate that Event's handler on the Saga with `@EndSaga`. The Saga's Life Cycle will be ended after the invocation of the handler. Alternatively, you can call `SagaLifecycle.end()` from inside the Saga to end the life cycle. This allows you to conditionally end the Saga.

### Event Handling

Event Handling in a Saga is quite comparable to that of a regular Event Listener. The same rules for method and parameter resolution are valid here. There is one major difference, though. While there is a single instance of an Event Listener that deals with all incoming events, multiple instances of a Saga may exist, each interested in different Events. For example, a Saga that manages a transaction around an Order with Id "1" will not be interested in Events regarding Order "2", and vice versa.

Instead of publishing all Events to all Saga instances \(which would be a complete waste of resources\), Axon will only publish Events containing properties that the Saga has been associated with. This is done using `AssociationValue`s. An `AssociationValue` consists of a key and a value. The key represents the type of identifier used, for example "orderId" or "order". The value represents the corresponding value, "1" or "2" in the previous example.

The order in which `@SagaEventHandler` annotated methods are evaluated is identical to that of `@EventHandler` methods \(see [Annotated Event Handler](event-handling.md#defining-event-handlers)\). A method matches if the parameters of the handler method match the incoming Event, and if the saga has an association with the property defined on the handler method.

The `@SagaEventHandler` annotation has two attributes, of which `associationProperty` is the most important one. This is the name of the property on the incoming Event that should be used to find associated Sagas. The key of the association value is the name of the property. The value is the value returned by property's getter method.

For example, consider an incoming Event with a method "`String getOrderId()`", which returns "123". If a method accepting this Event is annotated with `@SagaEventHandler(associationProperty="orderId")`, this Event is routed to all Sagas that have been associated with an `AssociationValue` with key "orderId" and value "123". This may either be exactly one, more than one, or even none at all.

Sometimes, the name of the property you want to associate with is not the name of the association you want to use. For example, you have a Saga that matches Sell orders against Buy orders, you could have a Transaction object that contains the "buyOrderId" and a "sellOrderId". If you want the saga to associate that value as "orderId", you can define a different keyName in the `@SagaEventHandler` annotation. It would then become `@SagaEventHandler(associationProperty="sellOrderId", keyName="orderId")`

### Managing associations

When a Saga manages a transaction across multiple domain concepts, such as Order, Shipment, Invoice, etc, that Saga needs to be associated with instances of those concepts. An association requires two parameters: the key, which identifies the type of association \(Order, Shipment, etc\) and a value, which represents the identifier of that concept.

Associating a Saga with a concept is done in several ways. First of all, when a Saga is newly created when invoking a `@StartSaga` annotated Event Handler, it is automatically associated with the property identified in the `@SagaEventHandler` method. Any other association can be created using the `SagaLifecycle.associateWith(String key, String/Number value)` method. Use the `SagaLifecycle.removeAssociationWith(String key, String/Number value)` method to remove a specific association.

> Note
>
> The API to associate domain concepts within a Saga intentionally only allows a `String` or a `Number` as the identifying value, since a `String` representation of the identifier is required for the association value entry which is stored. Using simple identifier values in the API with a straightforward `String` representation is by design, as a `String` column entry in the database makes the comparison between database engines simpler. It is thus intentionally that there is no `associateWith(String, Object)` for example, as the result of an `Object#toString()` call might provide unwieldy identifiers.

Imagine a Saga that has been created for a transaction around an Order. The Saga is automatically associated with the Order, as the method is annotated with `@StartSaga`. The Saga is responsible for creating an Invoice for that Order, and tell Shipping to create a Shipment for it. Once both the Shipment have arrived and the Invoice has been paid, the transaction is completed and the Saga is closed.

Here is the code for such a Saga:

```java
public class OrderManagementSaga {

    private boolean paid = false;
    private boolean delivered = false;
    @Inject
    private transient CommandGateway commandGateway;

    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCreatedEvent event) {
        // client generated identifiers
        ShippingId shipmentId = createShipmentId();
        InvoiceId invoiceId = createInvoiceId();
        // associate the Saga with these values, before sending the commands
        SagaLifecycle.associateWith("shipmentId", shipmentId);
        SagaLifecycle.associateWith("invoiceId", invoiceId);
        // send the commands
        commandGateway.send(new PrepareShippingCommand(...));
        commandGateway.send(new CreateInvoiceCommand(...));
    }

    @SagaEventHandler(associationProperty = "shipmentId")
    public void handle(ShippingArrivedEvent event) {
        delivered = true;
        if (paid) { SagaLifecycle.end(); }
    }

    @SagaEventHandler(associationProperty = "invoiceId")
    public void handle(InvoicePaidEvent event) {
        paid = true;
        if (delivered) { SagaLifecycle.end(); }
    }

    // ...
}
```

By allowing clients to generate an identifier, a Saga can be easily associated with a concept, without the need to a request-response type command. We associate the event with these concepts before publishing the command. This way, we are guaranteed to also catch events generated as part of this command. This will end this saga once the invoice is paid and the shipment has arrived.

### Keeping track of Deadlines

It is easy to make a Saga take action when something happens. After all, there is an Event to notify the Saga. But what if you want your Saga to do something when _nothing_ happens? That's what deadlines are used for. In invoices, that's typically several weeks, while the confirmation of a credit card payment should occur within a few seconds.

In Axon, you can use an `EventScheduler` to schedule an Event for publication. In the example of an Invoice, you'd expect that invoice to be paid within 30 days. A Saga would, after sending the `CreateInvoiceCommand`, schedule an `InvoicePaymentDeadlineExpiredEvent` to be published in 30 days. The EventScheduler returns a `ScheduleToken` after scheduling an Event. This token can be used to cancel the schedule, for example when a payment of an Invoice has been received.

Axon provides two `EventScheduler` implementations: a pure Java one and one using Quartz 2 as a backing scheduling mechanism.

This pure-Java implementation of the `EventScheduler` uses a `ScheduledExecutorService` to schedule Event publication. Although the timing of this scheduler is very reliable, it is a pure in-memory implementation. Once the JVM is shut down, all schedules are lost. This makes this implementation unsuitable for long-term schedules.

The `SimpleEventScheduler` needs to be configured with an `EventBus` and a `SchedulingExecutorService` \(see the static methods on the `java.util.concurrent.Executors` class for helper methods\).

The `QuartzEventScheduler` is a more reliable and enterprise-worthy implementation. Using Quartz as underlying scheduling mechanism, it provides more powerful features, such as persistence, clustering and misfire management. This means Event publication is guaranteed. It might be a little late, but it will be published.

It needs to be configured with a Quartz `Scheduler` and an `EventBus`. Optionally, you may set the name of the group that Quartz jobs are scheduled in, which defaults to "AxonFramework-Events".

One or more components will be listening for scheduled Events. These components might rely on a Transaction being bound to the Thread that invokes them. Scheduled Events are published by Threads managed by the `EventScheduler`. To manage transactions on these threads, you can configure a `TransactionManager` or a `UnitOfWorkFactory` that creates a Transaction Bound Unit of Work.

> **Note**
>
> Spring users can use the `QuartzEventSchedulerFactoryBean` or `SimpleEventSchedulerFactoryBean` for easier configuration. It allows you to set the PlatformTransactionManager directly.

### Injecting Resources

Sagas generally do more than just maintaining state based on Events. They interact with external components. To do so, they need access to the Resources necessary to address to components. Usually, these resources aren't really part of the Saga's state and shouldn't be persisted as such. But once a Saga is reconstructed, these resources must be injected before an Event is routed to that instance.

For that purpose, there is the `ResourceInjector`. It is used by the `SagaRepository` to inject resources into a Saga. Axon provides a `SpringResourceInjector`, which injects annotated fields and methods with Resources from the Application Context, and a `SimpleResourceInjector`, which injects resources that have been registered with it into `@Inject` annotated methods and fields.

> **Tip**
>
> Since resources should not be persisted with the Saga, make sure to add the `transient` keyword to those fields. This will prevent the serialization mechanism to attempt to write the contents of these fields to the repository. The repository will automatically re-inject the required resources after a Saga has been deserialized.

The `SimpleResourceInjector` allows for a pre-specified collection of resources to be injected. It scans the \(setter\) methods and fields of a Saga to find ones that are annotated with `@Inject`.

When using the Configuration API, Axon will default to the `ConfigurationResourceInjector`. It will inject any resource available in the Configuration. Components like the `EventBus`, `EventStore`, `CommandBus` and `CommandGateway` are available by default, but you can also register your own components using `configurer.registerComponent()`.

The `SpringResourceInjector` uses Spring's dependency injection mechanism to inject resources into an aggregate. This means you can use setter injection or direct field injection if you require. The method or field to be injected needs to be annotated in order for Spring to recognize it as a dependency, for example with `@Autowired`.

## Saga Infrastructure

Events need to be redirected to the appropriate Saga instances. To do so, some infrastructure classes are required. The most important components are the `SagaManager` and the `SagaRepository`.

### Saga Manager

Like any component that handles Events, the processing is done by an Event Processor. However, since Sagas aren't singleton instances handling Events, but have individual life cycles, they need to be managed.

Axon supports life cycle management through the `AnnotatedSagaManager`, which is provided to an Event Processor to perform the actual invocation of handlers. It is initialized using the type of the Saga to manage, as well as a SagaRepository where Sagas of that type can be stored and retrieved. A single `AnnotatedSagaManager` can only manage a single Saga type.

When using the Configuration API, Axon will use sensible defaults for most components. However, it is highly recommended to define a `SagaStore` implementation to use. The `SagaStore` is the mechanism that 'physically' stores the Saga instances somewhere. The `AnnotatedSagaRepository` \(the default\) uses the `SagaStore` to store and retrieve Saga instances as they are required.

{% codetabs name="Axon Configuration API", type="java" -%}
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.registerModule(
        SagaConfiguration.subscribingSagaManager(MySaga.class)
                         // Axon defaults to an in-memory SagaStore, defining another is recommended
                         .configureSagaStore(c -> new JpaSagaStore(...)));

// alternatively, it is possible to register a single SagaStore for all Saga types:
configurer.registerComponent(SagaStore.class, c -> new JpaSagaStore(...));
{%- language name="Spring Boot AutoConfiguration", type="java" -%}
@Saga(sagaStore = "mySagaStore")
public class MySaga {...}
...
// somewhere in configuration
@Bean
public SagaStore mySagaStore() {
    return new MongoSagaStore(...); // default is JpaSagaStore
}
{%- endcodetabs %}

### Saga Repository and Saga Store

The `SagaRepository` is responsible for storing and retrieving Sagas, for use by the `SagaManager`. It is capable of retrieving specific Saga instances by their identifier as well as by their Association Values.

There are some special requirements, however. Since concurrency handling in Sagas is a very delicate procedure, the repository must ensure that for each conceptual Saga instance \(with equal identifier\) only a single instance exists in the JVM.

Axon provides the `AnnotatedSagaRepository` implementation, which allows the lookup of Saga instances while guaranteeing that only a single instance of the Saga is accessed at the same time. It uses a `SagaStore` to perform the actual persistence of Saga instances.

The choice for the implementation to use depends mainly on the storage engine used by the application. Axon provides the `JdbcSagaStore`, `InMemorySagaStore`, `JpaSagaStore` and `MongoSagaStore`.

In some cases, applications benefit from caching Saga instances. In that case, there is a `CachingSagaStore` which wraps another implementation to add caching behavior. Note that the `CachingSagaStore` is a write-through cache, which means save operations are always immediately forwarded to the backing Store, to ensure data safety.

#### JpaSagaStore

The `JpaSagaStore` uses JPA to store the state and Association Values of Sagas. Sagas themselves do not need any JPA annotations; Axon will serialize the sagas using a `Serializer` \(similar to Event serialization, you can choose between an `XStreamSerializer`, `JacksonSerializer` or `JavaSerializer`, which can be set by configuring the 'Default `Serializer`' in your application. For more detail, check the [Advanced Customizations](../part-iv-advanced-tuning/advanced-customizations.md#serializers) section\).

The `JpaSagaStore` is configured with an `EntityManagerProvider`, which provides access to an `EntityManager` instance to use. This abstraction allows for the use of both application managed and container managed `EntityManager`s. Optionally, you can define the serializer to serialize the Saga instances with. Axon defaults to the `XStreamSerializer`.

#### JdbcSagaStore

The `JdbcSagaStore` uses plain JDBC to store stage instances and their association values. Similar to the `JpaSagaStore`, Saga instances don't need to be aware of how they are stored. They are serialized using a Serializer.

The `JdbcSagaStore` is initialized with either a `DataSource` or a `ConnectionProvider`. While not required, when initializing with a `ConnectionProvider`, it is recommended to wrap the implementation in a `UnitOfWorkAwareConnectionProviderWrapper`. It will check the current Unit of Work for an already open database connection, to ensure that all activity within a unit of work is done on a single connection.

Unlike JPA, the JdbcSagaRepository uses plain SQL statement to store and retrieve information. This may mean that some operations depend on the Database specific SQL dialect. It may also be the case that certain Database vendors provide non-standard features that you would like to use. To allow for this, you can provide your own `SagaSqlSchema`. The `SagaSqlSchema` is an interface that defines all the operations the repository needs to perform on the underlying database. It allows you to customize the SQL statement executed for each one of them. The default is the `GenericSagaSqlSchema`. Other implementations available are `PostgresSagaSqlSchema`, `Oracle11SagaSqlSchema` and `HsqlSagaSchema`.

#### MongoSagaStore

The `MongoSagaStore` stores the Saga instances and their associations in a MongoDB database. The `MongoSagaStore` stores all sagas in a single Collection in a MongoDB database. Per Saga instance, a single document is created.

The `MongoSagaStore` also ensures that at any time, only a single Saga instance exists for any unique Saga in a single JVM. This ensures that no state changes are lost due to concurrency issues.

The `MongoSagaStore` is initialized using a `MongoTemplate` and optionally a `Serializer`. The `MongoTemplate` provides a reference to the collection to store the Sagas in. Axon provides the `DefaultMongoTemplate`, which takes the `MongoClient` instance as well as the database name and name of the collection to store the sagas in. The database name and collection name may be omitted. In that case, they default to "axonframework" and "sagas", respectively.

### Caching

If a database backed Saga Storage is used, saving and loading Saga instances may be a relatively expensive operation. Especially in situations where the same Saga instance is invoked multiple times within a short time span, a cache can be beneficial to the application's performance.

Axon provides the `CachingSagaStore` implementation. It is a `SagaStore` that wraps another one, which does the actual storage. When loading Sagas or Association Values, the `CachingSagaStore` will first consult its caches, before delegating to the wrapped repository. When storing information, all calls are always delegated, to ensure that the backing storage always has a consistent view on the Saga's state.

To configure caching, simply wrap any `SagaStore` in a `CachingSagaStore`. The constructor of the `CachingSagaStore` takes three parameters: the repository to wrap and the caches to use for the Association Values and Saga instances, respectively. The latter two arguments may refer to the same cache, or to different ones. This depends on the eviction requirements of your specific application.

