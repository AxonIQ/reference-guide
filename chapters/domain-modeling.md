Domain Modeling
===============

In a CQRS-based application, a Domain Model (as defined by Eric Evans and Martin Fowler) can be a very powerful mechanism to harness the complexity involved in the validation and execution of state changes. Although a typical Domain Model has a great number of building blocks, two of them play a major role when applied to CQRS: the Event and the Aggregate.

The following sections will explain the role of these building blocks and how to implement them using the Axon Framework.

Events
======

Events are objects that describe something that has occurred in the application. A typical source of events is the Aggregate. When something important has occurred within the aggregate, it will raise an Event. In Axon Framework, Events can be any object. You are highly encouraged to make sure all events are serializable.

When Events are dispatched, Axon wraps them in a Message. The actual type of Message used depends on the origin of the Event. When an Event is raised by an Aggregate, it is wrapped in a `DomainEventMessage` (which extends `EventMessage`). All other Events are wrapped in an `EventMessage.` The `EventMessage` contains a unique Identifier for the event, as well as a Timestamp and Meta Data. The `DomainEventMessage` additionally contains the identifier of the aggregate that raised the Event and the sequence number which allows the order of events to be reproduced.

Even though the DomainEventMessage contains a reference to the Aggregate Identifier, you should always include that identifier in the actual Event itself as well. The identifier in the DomainEventMessage is meant for the EventStore and may not always provide a reliable value for other purposes.

The original Event object is stored as the Payload of an EventMessage. Next to the payload, you can store information in the Meta Data of an Event Message. The intent of the Meta Data is to store additional information about an Event that is not primarily intended as business information. Auditing information is a typical example. It allows you to see under which circumstances an Event was raised, such as the User Account that triggered the processing, or the name of the machine that processed the Event.

> **Note**
>
> In general, you should not base business decisions on information in the meta-data of event messages. If that is the case, you might have information attached that should really be part of the Event itself instead. Meta-data is typically used for auditing and tracing.

Although not enforced, it is good practice to make domain events immutable, preferably by making all fields final and by initializing the event within the constructor.

> **Note**
>
> Although Domain Events technically indicate a state change, you should try to capture the intention of the state in the event, too. A good practice is to use an abstract implementation of a domain event to capture the fact that certain state has changed, and use a concrete sub-implementation of that abstract class that indicates the intention of the change. For example, you could have an abstract `AddressChangedEvent`, and two implementations `ContactMovedEvent` and `AddressCorrectedEvent` that capture the intent of the state change. Some listeners don't care about the intent (e.g. database updating event listeners). These will listen to the abstract type. Other listeners do care about the intent and these will listen to the concrete subtypes (e.g. to send an address change confirmation email to the customer).
>
> ![ "Adding intent to events](../images/state-change-intent.png)

When dispatching an Event on the Event Bus, you will need to wrap it in an (Domain) Event Message. The `GenericEventMessage` is an implementation that allows you to wrap your Event in a Message. You can use the constructor, or the static `asEventMessage()` method. The latter checks whether the given parameter doesn't already implement the `Message` interface. If so, it is either returned directly (if it implements `EventMessage`,) or it returns a new `GenericEventMessage` using the given `Message`'s payload and Meta Data.

Aggregate
=========
An Aggregate is an entity or group of entities that is always kept in a consistent state. The aggregate root is the object on top of the aggregate tree that is responsible for maintaining this consistent state.

> **Note**
>
> The term "Aggregate" refers to the aggregate as defined by Evans in Domain Driven Design:
>
> “A cluster of associated objects that are treated as a unit for the purpose of data changes. External references are restricted to one member of the Aggregate, designated as the root. A set of consistency rules applies within the Aggregate's boundaries.”

For example, a "Contact" aggregate could contain two entities: Contact and Address. To keep the entire aggregate in a consistent state, adding an address to a contact should be done via the Contact entity. In this case, the Contact entity is the appointed aggregate root.

In Axon, aggregates are identified by an Aggregate Identifier. This may be any object, but there are a few guidelines for good implementations of identifiers. Identifiers must:

-   implement equals and hashCode to ensure good equality comparison with other instances,

-   implement a toString() method that provides a consistent result (equal identifiers should provide an equal toString() result), and

-   preferably be serializable.

The test fixtures (see [Testing](testing.md)) will verify these conditions and fail a test when an Aggregate uses an incompatible identifier. Identifiers of type `String`, `UUID` and the numeric types are always suitable.

> **Note**
>
> It is considered a good practice to use randomly generated identifiers, as opposed to sequenced ones. Using a sequence drastically reduces scalability of your application, since machines need to keep each other up-to-date of the last used sequence numbers. The chance of collisions with a UUID is very slim (a chance of 10<sup>−15</sup>, if you generate 8.2 × 10 <sup>11</sup> UUIDs).
>
> Furthermore, be careful when using functional identifiers for aggregates. They have a tendency to change, making it very hard to adapt your application accordingly.

Aggregate implementations
-------------------------------
An Aggregate is always accessed through a single Entity, called the Aggregate Root. Usually, the name of this Entity is the same as that of the Aggregate entirely. For example, an Order Aggregate may consist of an Order entity, which references several OrderLine entities. Order and Orderline together, form the Aggregate.

An Aggregate is a regular object, which contains state and methods to alter that state. Although not entirely correct according to CQRS principles, it is also possible to expose the state of the aggregate through accessor methods.

An Aggregate Root must declare a field that contains the Aggregate identifier. This identifier must be initialized at the latest when the first Event is published. This identifier field must be annotated by the `@AggregateIdentifier` annotation.
If you use JPA and have JPA annotations on the aggregate, Axon can also use the `@Id` annotation provided by JPA.

Aggregates may use the `AggregateLifecycle.apply()` method to register events for publication. Unlike the `EventBus`, where messages need to be wrapped in an EventMessage, `apply()` allows you to pass in the payload object directly.

``` java
@Entity // Mark this aggregate as a JPA Entity
public class MyAggregate {
    
    @Id // When annotating with JPA @Id, the @AggregateIdentifier annotation is not necessary
    private String id;
    
    // fields containing state...
    
    public MyAggregate(CreateMyAggregateCommand command) {
        // ... update state
        apply(new MyAggregateCreatedEvent(...));
    }
    
    // constructor needed by JPA
    protected MyAggregateRoot() {
    }
}
```

Event sourced aggregates
------------------------
Besides storing the current state of an Aggregate, it is also possible to rebuild the state of an Aggregate based on the Events that it has published in the past. For this to work, all state changes must be represented by an Event.

For the major part, Event Sourced Aggregates are similar to 'regular' aggregates: they must declare an identifier and can use the `apply` method to publish Events. However, state changes in Event Sourced Aggregates (i.e. any change of a Field value) must be performed in an `@EventSourcedHandler` annotated method. This includes setting the Aggregate Identifier.

Note that the Aggregate Identifier must be set in the `@EventSourcingHandler` of the very first Event published by the Aggregate. This is usually the creation Event.

The Aggregate Root of an Event Sourced Aggregate must also contain a no-arg constructor. Axon Framework uses this constructor to create an empty Aggregate instance before initialize it using past Events. Failure to provide this constructor will result in an Exception when loading the Aggregate.

``` java
public class MyAggregateRoot {

    @AggregateIdentifier
    private String aggregateIdentifier;
    
    // fields containing state...

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

`@EventSourcingHandler` annotated methods are resolved using specific rules. These rules are the same for the `@EventHandler` annotated methods, and are thoroughly explained in [Annotated Event Handler](event-listener.md#annotated-event-handler).

> **Note**
>
> Event handler methods may be private, as long as the security settings of the JVM allow the Axon Framework to change the accessibility of the method. This allows you to clearly separate the public API of your aggregate, which exposes the methods that generate events, from the internal logic, which processes the events.
>
> Most IDE's have an option to ignore "unused private method" warnings for methods with a specific annotation. Alternatively, you can add an `@SuppressWarnings("UnusedDeclaration")` annotation to the method to make sure you don't accidentally delete an Event handler method.

In some cases, especially when aggregate structures grow beyond just a couple of entities, it is cleaner to react on events being published in other entities of the same aggregate. However, since Event Handler methods are also invoked when reconstructing aggregate state, special precautions must be taken.

It is possible to `apply()` new events inside an Event Sourcing Handler method. This makes it possible for an Entity B to apply an event in reaction to Entity A doing something. Axon will ignore the apply() invocation when replaying historic events. Do note that, in this case, the Event of the inner `apply()` invocation is only published to the Entities after all Entities have received the first Event. If more events need to be published, based on the state of an entity after applying an inner event, use `apply(...).andThenApply(...)`

You can also use the static `AggregateLifecycle.isLive()` method to check whether the aggregate is 'live'. Basically, an aggregate is considered live if it has finished replaying historic events. While replaying these events, isLive() will return false. Using this `isLive()` method, you can perform activity that should only be done when handling newly generated events.

Complex Aggregate structures
----------------------------
Complex business logic often requires more than what an aggregate with only an aggregate root can provide. In that case, it is important that the complexity is spread over a number of entities within the aggregate. When using event sourcing, not only the aggregate root needs to use events to trigger state transitions, but so does each of the entities within that aggregate.

> ** Note **
> A common misinterpretation of the rule that Aggregates should not expose state is that none of the Entities should contain any property accessor methods. This is not the case. In fact, an Aggregate will probably benefit a lot if the entities *within* the aggregate expose state to the other entities in that same aggregate. However, is is recommended not to expose the state *outside* of the Aggregate.

Axon provides support for event sourcing in complex aggregate structures. Entities are, just like the Aggregate Root, simple objects. The field that declares the child entity must be annotated with `@AggregateMember`. This annotation tells Axon that the annotated field contains a class that should be inspected for Command and Event Handlers.

When an Entity (including the Aggregate Root) applies an Event, it is handled by the Aggregate Root first, and then bubbles down through all `@AggregateMember` annotated fields to its child entities.

Fields that (may) contain child entities must be annotated with `@AggregateMember`. This annotation may be used on a number of field types:

-   the Entity Type, directly referenced in a field;

-   inside fields containing an `Iterable` (which includes all collections, such as `Set`, `List`, etc);

-   inside the values of fields containing a `java.util.Map`

When Entities are declared in their parent in a `Collection` or `Map`, the entities must have an identifier annotated with `@EntityId`. This identifier is used to identify the Entity instance inside the Collection to which a Command should be routed. See [Command Handling](command-handling.md) for more details on Command routing.
