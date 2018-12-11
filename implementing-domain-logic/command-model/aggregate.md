# Aggregate

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