# 1.2.1 Command handling

In a CQRS-based application, a Domain Model \(as defined by Eric Evans and [Martin Fowler](https://martinfowler.com/bliki/CQRS.html)\) can be a very powerful mechanism to harness the complexity involved in the validation and execution of state changes. Although a typical domain model has a great number of building blocks, one of them plays a dominant role when applied to command processing in CQRS: the Aggregate.

A state change within an application starts with a Command. A command is a combination of expressed intent \(which describes what you want done\) as well as the information required to undertake action based on that intent. The command model is used to process the incoming command, to validate it and define the outcome. Within this model, a command handler is responsible for handling commands of a certain type and taking action based on the information contained inside it.

## Aggregate

An Aggregate is an entity or group of entities that is always kept in a consistent state. The Aggregate Root is the object on top of the aggregate tree that is responsible for maintaining this consistent state. This makes the aggregate the prime building block for implementing a command model in any CQRS based application.

> **Note**
>
> The term "Aggregate" refers to the aggregate as defined by Evans in Domain-Driven Design:
>
> “A cluster of associated objects that are treated as a unit for the purpose of data changes. External references are restricted to one member of the Aggregate, designated as the root. A set of consistency rules applies within the Aggregate's boundaries.”

For example, a "Contact" aggregate could contain two entities: Contact and Address. To keep the entire aggregate in a consistent state, adding an address to a contact should be done via the Contact entity. In this case, the Contact entity is the appointed aggregate root.

In Axon, aggregates are identified by an Aggregate Identifier. This may be any object, but there are a few guidelines for good implementations of identifiers. Identifiers must:

* implement `equals` and `hashCode` to ensure good equality comparison with other instances,
* implement a `toString()` method that provides a consistent result \(equal identifiers should provide an equal toString\(\) result\), and
* preferably be `Serializable`.

The test fixtures \(see [Testing](testing.md)\) will verify these conditions and fail a test when an aggregate uses an incompatible identifier. Identifiers of type `String`, `UUID` and the numeric types are always suitable. Do **not** use primitive types as identifiers, as they do not allow for lazy initialization. Axon may, in some circumstances, falsely assume the default value of a primitive to be the value of the identifier.

> **Note**
>
> It is considered a good practice to use randomly generated identifiers, as opposed to sequenced ones. Using a sequence drastically reduces scalability of your application, since machines need to keep each other up-to-date of the last used sequence numbers. The chance of collisions with a UUID is very slim \(a chance of 10−15, if you generate 8.2 × 10 11 UUIDs\).
>
> Furthermore, be careful when using functional identifiers for aggregates. They have a tendency to change, making it very hard to adapt your application accordingly.

## Aggregate implementations

An aggregate is always accessed through a single entity, called the Aggregate Root. Usually, the name of this entity is the same as that of the aggregate entirely. For example, an Order aggregate may consist of an Order entity, which references several OrderLine entities. Order and Orderline together, form the aggregate.

An aggregate is a regular object, which contains state and methods to alter that state. Although, not entirely correct according to CQRS principles, it is also possible to expose the state of the aggregate through accessor methods.

An aggregate root must declare a field that contains the aggregate identifier. This identifier must be initialized at the latest when the first event is published. This identifier field must be annotated by the `@AggregateIdentifier` annotation. If you use JPA and have JPA annotations on the aggregate, Axon can also use the `@Id` annotation provided by JPA.

Aggregates may use the `AggregateLifecycle.apply()` method to register events for publication. Unlike the `EventBus`, where messages need to be wrapped in an `EventMessage`, `apply()` allows you to pass in the payload object directly.

```java
import static org.axonframework.commandhandling.model.AggregateLifecycle.apply;

@Entity // Mark this aggregate as a JPA Entity
public class MyAggregate {

    @Id // When annotating with JPA @Id, the @AggregateIdentifier annotation 
        // is not necessary
    private String id;

    // fields containing state...

    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        // ... update state
        apply(new MyAggregateCreatedEvent(...));
    }

    // constructor needed by JPA
    protected MyAggregate() {
    }
}
```

Entities within an Aggregate can listen to the events the Aggregate publishes, by defining an `@EventHandler` annotated method. These methods will be invoked when an EventMessage is published \(before any external handlers are published\).

## Event sourced aggregates

Besides storing the current state of an aggregate, it is also possible to rebuild the state of an aggregate based on the events that it has published in the past. For this to work, all state changes must be represented by an event.

For the major part, event sourced aggregates are similar to 'regular' aggregates: they must declare an identifier and can use the `apply()` method to publish Events. However, state changes in event sourced aggregates \(i.e. any change of a Field value\) must be _exclusively_ performed in an `@EventSourcingHandler` annotated method. This includes setting the aggregate Identifier.

Note that the aggregate identifier must be set in the `@EventSourcingHandler` of the very first Event published by the aggregate. This is usually the creation event.

The aggregate root of an event sourced aggregate must also contain a no-arg constructor. Axon Framework uses this constructor to create an empty aggregate instance before initializing it using past Events. Failure to provide this constructor will result in an exception when loading the aggregate.

```java
public class MyAggregateRoot {

    @AggregateIdentifier
    private String aggregateIdentifier;

    // fields containing state...

    @CommandHandler
    public MyAggregateRoot(CreateMyAggregate cmd) {
        apply(new MyAggregateCreatedEvent(cmd.getId()));
    }

    // constructor needed for reconstruction
    protected MyAggregateRoot() {
    }

    @EventSourcingHandler
    private void handleMyAggregateCreatedEvent(MyAggregateCreatedEvent event) {
        // make sure identifier is always initialized properly
        this.aggregateIdentifier = event.getMyAggregateIdentifier();

        // ... update state
    }
}
```

`@EventSourcingHandler` annotated methods are resolved using specific rules.

These rules are the same for the `@EventHandler` annotated methods, and are thoroughly explained in [Annotated Event Handler](event-handling.md#defining-event-handlers).

> **Note**
>
> Event handler methods may be private, as long as the security settings of the JVM allow the Axon Framework to change the accessibility of the method. This allows you to clearly separate the public API of your aggregate, which exposes the methods that generate events, from the internal logic, which processes the events.
>
> Most IDE's have an option to ignore "unused private method" warnings for methods with a specific annotation. Alternatively, you can add an `@SuppressWarnings("UnusedDeclaration")` annotation to the method to make sure you do not accidentally delete an event handler method.

In some cases, especially when aggregate structures grow beyond just a couple of entities, it is cleaner to react on events being published in other entities of the same aggregate. However, since Event Handler methods are also invoked when reconstructing aggregate state, special precautions must be taken.

It is possible to `apply()` new events inside an event sourcing handler method. This makes it possible for an entity B to apply an event in reaction to entity A doing something. Axon will ignore the `apply()`invocation when replaying historic events. Do note that, in this case, the Event of the inner `apply()` invocation is only published to the entities after all entities have received the first event. If more events need to be published, based on the state of an entity after applying an inner event, use `apply(...).andThenApply(...)`

You can also use the static `AggregateLifecycle.isLive()` method to check whether the aggregate is 'live'. Basically, an aggregate is considered live if it has finished replaying historic events. While replaying these events, `isLive()` will return false. Using this `isLive()` method, you can perform activity that should only be done when handling newly generated events.

## Spawning a new Aggregate

Instantiating new Aggregates is done by issuing a creation command, which for example might originate from an Event Handling Component \(Saga or Event Processor\). But in some cases it might be beneficial to create an Aggregate from another Aggregate. Prior to version 3.3, instantiating an Aggregate as a follow up form another Aggregate had to be orchestrated via an Event Handling Component. Version 3.3 introduces functionality to create a new Aggregate from another Aggregate. To this end, the `AggregateLifecycle` introduces the static method `createNew()`.

Consider a case where you have `AggregateA` defined like this:

```java
public class AggregateA {
    ...            
    public AggregateA(String id) {
        // apply the creation event
    }
    ...
}
```

We would like to create this Aggregate as a consequence of handling a command in `AggregateB`, like so:

```java
public class AggregateB {
    ...
    @CommandHandler
    public void AggregateB(SomeAggregateBCommand command) {
        AggregateLifecycle.createNew(AggregateA.class, () -> new AggregateA(/* provide the id for AggregateA */)); // (1)
    }
    ...
}
```

\(1\) The first parameter of the `AggregateLifecycle#createNew()` method is the type of Aggregate to be created. The second parameter is the factory method - the method to be used in order to instantiate the desired Aggregate.

> **Note** Creation of a new Aggregate should be done in a Command Handling function rather than in an Event Handling function \(given the usage of Event Sourced Aggregate\). Rationale: we do not want to create new Aggregates when we are sourcing a given Aggregate - previously created aggregate will be Event Sourced based on its events. However, if you try to create a new Aggregate while Axon is replaying events, an `UnsupportedOperationException` will be thrown.

## Complex aggregate structures

Complex business logic often requires more than what an aggregate with only an aggregate root can provide. In that case, it is important that the complexity is spread over a number of entities within the aggregate. When using event sourcing, not only the aggregate root needs to use events to trigger state transitions, but so does each of the entities within that aggregate.

> **Note** 
>
> A common misinterpretation of the rule that aggregates should not expose state is that none of the entities should contain any property accessor methods. This is not the case. In fact, an aggregate will probably benefit a lot if the entities _within_ the aggregate expose state to the other entities in that same aggregate. However, is is recommended not to expose the state _outside_ of the aggregate.

Axon provides support for event sourcing in complex aggregate structures. Entities are, just like the aggregate root, simple objects. The field that declares the child entity must be annotated with `@AggregateMember`. This annotation tells Axon that the annotated field contains a class that should be inspected for command and event handlers.

When an entity \(including the aggregate root\) applies an event, it is handled by the aggregate root first, and then bubbles down through all `@AggregateMember` annotated fields to its child entities.

> **Note** There is a way to filter the entities which would handle an event applied by the Aggregate Root. This can be achieved by using `eventForwardingMode` attribute of `@AggregateMember` annotation. By default, an event is propagated to **all** child entities. An event can be blocked using `ForwardNone` event forwarding mode \(see listing below\).
>
> ```java
> public class MyAggregate {
>    ...
>    @AggregateMember(eventForwardingMode = ForwardNone.class)
>    private MyEntity myEntity;
>    ...
> }
> ```
>
> If you want to forward an event to the entity only in a case when an event message has matching entity identifier use `ForwardMatchingInstances` event forwarding mode. Entity identifier matching will be done based on specified `routingKey` on `@AggregateMember` annotation. If `routingKey` is not specified on `@AggregateMember` annotation, matching will be done based on `routingKey` attribute on `@EntityId` annotation. If `routingKey` is not specified on `@EntityId` annotation matching will be done based on field name of entity identifier. Let's take a look at example on how to define `ForwardMatchingInstances` event forwarding mode with specifying a routing key for the entity identifier:
>
> ```java
> public class MyAggregate {
>    ...
>    @AggregateMember(eventForwardingMode = ForwardMatchingInstances.class)
>    private MyEntity myEntity;
>    ...
> }
> ...
> public class MyEntity {
>    ...
>    @EntityId(routingKey = "myEntityId")
>    private String id;
>    ...
>    @EventSourcingHandler
>    public void on(MyEvent event) {
>        // handle event
>    }
> }
> ...
> public class MyEvent {
>    ...
>    private String myEntityId;
>    ...
> }
> ```

Fields that \(may\) contain child entities must be annotated with `@AggregateMember`. This annotation may be used on a number of field types:

* the entity type, directly referenced in a field
* inside fields containing an `Iterable` \(which includes all collections, such as `Set`, `List`, etc\)
* inside the values of fields containing a `java.util.Map`

### Handling commands in an Aggregate

It is recommended to define the command handlers directly in the aggregate that contains the state to process this command, as it is not unlikely that a command handler needs the state of that aggregate to do its job.

To define a command handler in an aggregate, simply annotate the command handling method with `@CommandHandler`. The rules for an `@CommandHandler` annotated method are the same as for any handler method. However, commands are not only routed by their payload. command nessages carry a name, which defaults to the fully qualified class name of the command object.

By default, `@CommandHandler` annotated methods allow the following parameter types:

* The first parameter is the payload of the command message. It may also be of type `Message` or `CommandMessage`, if the `@CommandHandler` annotation explicitly defined the name of the command the handler can process. By default, a command name is the fully qualified class name of the command its payload.
* Parameters annotated with `@MetaDataValue` will resolve to the metadata value with the key as indicated on the annotation. If `required` is `false` \(default\), `null` is passed when the metadata value is not present. If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.
* Parameters of type `MetaData` will have the entire `MetaData` of a `CommandMessage` injected.
* Parameters of type `UnitOfWork` get the current unit of work injected. This allows command handlers to register actions to be performed at specific stages of the Unit of Work, or gain access to the resources registered with it.
* Parameters of type `Message`, or `CommandMessage` will get the complete message, with both the payload and the metadata. This is useful if a method needs several metadata fields, or other properties of the wrapping Message.

In order for Axon to know which instance of an aggregate type should handle the command message, the property carrying the aggregate identifier in the command object must be annotated with `@TargetAggregateIdentifier`. The annotation may be placed on either the field or an accessor method \(e.g. a getter\).

Commands that create an aggregate instance do not need to identify the target aggregate identifier, although it is recommended to annotate the aggregate identifier on them as well.

If you prefer to use another mechanism for routing commands, the behavior can be overridden by supplying a custom `CommandTargetResolver`. This class should return the aggregate identifier and expected version \(if any\) based on a given command.

> **Note**
>
> When the `@CommandHandler` annotation is placed on an aggregate's constructor, the respective command will create a new instance of that aggregate and add it to the repository. Those commands do not require to target a specific aggregate instance. Therefore, those commands do not require any `@TargetAggregateIdentifier` or `@TargetAggregateVersion` annotations, nor will a custom `CommandTargetResolver` be invoked for these commands.
>
> When a command creates an aggregate instance, the callback for that command will receive the aggregate identifier when the command executed successfully.

```java
import static org.axonframework.commandhandling.model.AggregateLifecycle.apply;

public class MyAggregate {

    @AggregateIdentifier
    private String id;

    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        apply(new MyAggregateCreatedEvent(IdentifierFactory.getInstance().generateIdentifier()));
    }

    // no-arg constructor for Axon
    MyAggregate() {
    }

    @CommandHandler
    public void doSomething(DoSomethingCommand command) {
        // do something...
    }

    // code omitted for brevity. The event handler for MyAggregateCreatedEvent must set the id field
}

public class DoSomethingCommand {

    @TargetAggregateIdentifier
    private String aggregateId;

    // code omitted for brevity

}
```

Let's see how we can configure an Aggregate:


{% codetabs name="Axon Configuration API", type="java" -%}

Configurer configurer = ...
// to use defaults:
configurer.configureAggreate(MyAggregate.class);

// allowing customizations:
configurer.configureAggregate(
        AggregateConfigurer.defaultConfiguration(MyAggregate.class)
                           .configureCommandTargetResolver(c -> new CustomCommandTargetResolver())
);

{%- language name="Spring Boot AutoConfiguration", type="java" -%}

@Aggregate(commandTargetResolver = "customCommandTargetResolver")
public class MyAggregate {...}
...
// somewhere in configuration
@Bean
public CommandTargetResolver customCommandTargetResolver() {
    return new CustomCommandTargetResolver();
}

{%- endcodetabs %}

`@CommandHandler` annotations are not limited to the aggregate root. Placing all command handlers in the root will sometimes lead to a large number of methods on the aggregate root, while many of them simply forward the invocation to one of the underlying entities. If that is the case, you may place the `@CommandHandler` annotation on one of the underlying entities' methods. For Axon to find these annotated methods, the field declaring the entity in the aggregate root must be marked with `@AggregateMember`. Note that only the declared type of the annotated field is inspected for command handlers. If a field value is null at the time an incoming command arrives for that entity, an exception is thrown.

```java
public class MyAggregate {

    @AggregateIdentifier
    private String id;

    @AggregateMember
    private MyEntity entity;

    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        apply(new MyAggregateCreatedEvent(...);
    }

    // no-arg constructor for Axon
    MyAggregate() {
    }

    @CommandHandler
    public void doSomething(DoSomethingCommand command) {
        // do something...
    }

    // code omitted for brevity. The event handler for MyAggregateCreatedEvent must set the id field
    // and somewhere in the lifecycle, a value for "entity" must be assigned to be able to accept
    // DoSomethingInEntityCommand commands.
}

public class MyEntity {

    @CommandHandler
    public void handleSomeCommand(DoSomethingInEntityCommand command) {
        // do something
    }
}
```

> **Note**
>
> Note that each command must have exactly one handler in the aggregate. This means that you cannot annotate multiple entities \(either root nor not\) with `@CommandHandler`, that handle the same command type. In case you need to conditionally route a command to an entity, the parent of these entities should handle the command, and forward it based on the conditions that apply.
>
> The runtime type of the field does not have to be exactly the declared type. However, only the declared type of the `@AggregateMember` annotated field is inspected for `@CommandHandler` methods.

It is also possible to annotate collections and maps containing entities with `@AggregateMember`. In the latter case, the values of the map are expected to contain the entities, while the key contains a value that is used as their reference.

As a command needs to be routed to the correct instance, these instances must be properly identified. Their "id" field must be annotated with `@EntityId`. The property on the command that will be used to find the Entity that the message should be routed to, defaults to the name of the field that was annotated. For example, when annotating the field "myEntityId", the command must define a property with that same name. This means either a `getMyEntityId` or a `myEntityId()` method must be present. If the name of the field and the routing property differ, you may provide a value explicitly using `@EntityId(routingKey = "customRoutingProperty")`.

If no entity can be found in the annotated `Collection` or `Map`, Axon throws an `IllegalStateException` apparently, the aggregate is not capable of processing that command at that point in time.

> **Note**
>
> The field declaration for both the `Collection` or `Map`should contain proper generics to allow Axon to identify the type of Entity contained in the collection or map. If it is not possible to add the generics in the declaration \(e.g. because you're using a custom implementation which already defines generic types\), you must specify the type of entity used in the `entityType` property on the `@AggregateMember` annotation.

### External Command Handlers

In certain cases, it is not possible, or desired to route a command directly to an aggregate instance. In such case, it is possible to register a command handler object.

A command handler object is a simple \(regular\) object, which has `@CommandHandler` annotated methods. Unlike in the case of an aggregate, there is only a single instance of a command handler object, which handles all commands of the types it declares in its methods.

```java
public class MyAnnotatedHandler {

    @CommandHandler
    public void handleSomeCommand(SomeCommand command, @MetaDataValue("userId") String userId) {
        // whatever logic here
    }

    @CommandHandler(commandName = "myCustomCommand")
    public void handleCustomCommand(SomeCommand command) {
       // handling logic here
    }

}

// To register the annotated handlers to the command bus:
Configurer configurer = ...
configurer.registerCommandHandler(c -> new MyAnnotatedHandler());
```

### Returning results from command handlers

In some cases, the component dispatching a command needs information about the processing results of a command. A command handler method can return a value from its method. That value will be provided to the sender as the result of the command.

One exception is the `@CommandHandler` on an aggregate's constructor. In this case, instead of returning the return value of the method \(which is the Aggregate itself\), the value of the `@AggregateIdentifier` annotated field is returned instead

> **Note**
>
> While it is possible to return results from commands, it should be used sparsely. The intent of the command should never be in getting a value, as that would be an indication the message should be designed as a [Query Message ](https://docs.axoniq.io/reference-guide/1.2-domain-logic/query-handling)instead. A typical example for a Command result is the identifier of a newly created entity.

