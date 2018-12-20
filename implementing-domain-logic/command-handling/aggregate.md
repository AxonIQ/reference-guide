# Aggregate

This chapter will cover the basics on how to implement an ['Aggregate'](../../introduction/architecture-overview/ddd-cqrs-concepts.md#aggregates). 
For more specifics on what an Aggregate is it is beneficial to first read [DDD and CQRS Patterns](../../introduction/architecture-overview/ddd-cqrs-concepts.md),
 to get an initial grasp at what role the Aggregate plays.

## Basic Aggregate Structure

An Aggregate is a regular object, which contains state and methods to alter that state.
When creating the Aggregate object, you are effectively creating the 'Aggregate Root', typically carrying the name of the entire Aggregate.
For the purpose of this description the 'Gift Card' domain will be used, which brings us the `GiftCard` as the Aggregate (Root).
By default, Axon will configure your Aggregate as an 'Event Sourced' Aggregate (as described [here](../../introduction/architecture-overview/event-driven-microservices.md)).
Henceforth our basic `GiftCard` Aggregate structure will focus on the Event Sourcing approach:

```java
import org.axonframework.commandhandling.CommandHandler;
import org.axonframework.eventsourcing.EventSourcingHandler;
import org.axonframework.modelling.command.AggregateIdentifier;

import static org.axonframework.modelling.command.AggregateLifecycle.apply;

public class GiftCard {

    @AggregateIdentifier // 1.
    private String id;

    @CommandHandler // 2.
    public GiftCard(IssueCardCommand cmd) {
        // 3.
       apply(new CardIssuedEvent(cmd.getCardId(), cmd.getAmount()));
    }

    @EventSourcingHandler // 4.
    public void on(CardIssuedEvent evt) {
        id = evt.getCardId();
    }
    
    // 5.
    protected GiftCard() {
    }
}
```

There are a couple of noteworthy concepts from the given code snippets, marked with numbered Java comments referring to the following bullets: 

1. The `@AggregateIdentifier` is the external reference point to into the `GiftCard` Aggregate. 
This field is a hard requirement, as with out it Axon will not know to which Aggregate a given Command is targeted.
2. A `@CommandHandler` annotated constructor, or differently put the 'command handling constructor'. 
This annotation tells the framework that the given constructor is capable of handling the `IssueCardCommand`.
The `@CommandHandler` annotated functions are the place where you would put your decision-making/business logic. 
3. The static `AggregateLifecycle#apply(Object...)` is what is used when an Event Message should be published. 
Upon calling this function the provided `Object`s will be published as `EventMessage`s within the scope of the Aggregate they are applied in.
4. Using the `@EventSourcingHandler` is what tells the framework that the annotated function should be called when the Aggregate is 'sourced from its events'.
Note that the Aggregate Identifier **must** be set in the `@EventSourcingHandler` of the very first Event published by the aggregate. 
This is usually the creation event.
Lastly, `@EventSourcingHandler` annotated functions are resolved using specific rules.
These rules are the same for the `@EventHandler` annotated methods, and are thoroughly explained in [Annotated Event Handler](../event-handling/handling-events.md#handling-events).
5. A no-arg constructor, which is required by Axon.
Axon Framework uses this constructor to create an empty aggregate instance before initializing it using past Events. 
Failure to provide this constructor will result in an exception when loading the Aggregate.

> **Note**
>
> Event Handler methods may be private, as long as the security settings of the JVM allow the Axon Framework to change the accessibility of the method. 
> This allows you to clearly separate the public API of your Aggregate, which exposes the methods that generate events, from the internal logic, which processes the events.
>
> Most IDE's have an option to ignore "unused private method" warnings for methods with a specific annotation. 
> Alternatively, you can add an `@SuppressWarnings("UnusedDeclaration")` annotation to the method to make sure you do not accidentally delete an event handler method.

## Handling Commands in an Aggregate

It is recommended to define the command handlers directly in the aggregate that contains the state to process this command, as it is not unlikely that a command handler needs the state of that aggregate to do its job.

To define a command handler in an aggregate, simply annotate the command handling method with `@CommandHandler`. The rules for an `@CommandHandler` annotated method are the same as for any handler method. However, commands are not only routed by their payload. command nessages carry a name, which defaults to the fully qualified class name of the command object.

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

## Applying Events from Event Sourcing Handlers

In some cases, especially when the Aggregate structures grows beyond just a couple of Entities,
 it is cleaner to react on events being published in other Entities of the same Aggregate (multi Entity Aggregates are explained in more detail [here](multi-entity-aggregates.md)). 
However, since the Event Handling methods are also invoked when reconstructing Aggregate state, special precautions must be taken.

It is possible to `apply()` new events inside an Event Sourcing Handler method. 
This makes it possible for an Entity 'B' to apply an event in reaction to Entity 'A' doing something. 
Axon will ignore the `apply()`invocation when replaying historic events upon sourcing the given Aggregate. 
Do note that in the scenario where Event Messages are published from an Event Sourcing Handler,
 the Event of the inner `apply()` invocation is only published to the entities after all entities have received the first event. 
If more events need to be published, based on the state of an entity after applying an inner event, use `apply(...).andThenApply(...)`

## Returning results from Command Handlers

In some cases, the component dispatching a command needs information about the processing results of a command. A command handler method can return a value from its method. That value will be provided to the sender as the result of the command.

One exception is the `@CommandHandler` on an aggregate's constructor. In this case, instead of returning the return value of the method \(which is the Aggregate itself\), the value of the `@AggregateIdentifier` annotated field is returned instead

> **Note**
>
> While it is possible to return results from commands, it should be used sparsely. The intent of the command should never be in getting a value, as that would be an indication the message should be designed as a [Query Message ](https://docs.axoniq.io/reference-guide/1.2-domain-logic/query-handling)instead. A typical example for a Command result is the identifier of a newly created entity.

## Aggregate Lifecycle Operations

You can also use the static `AggregateLifecycle.isLive()` method to check whether the aggregate is 'live'. 
Basically, an aggregate is considered live if it has finished replaying historic events. 
While replaying these events, `isLive()` will return false. 
Using this `isLive()` method, you can perform activity that should only be done when handling newly generated events.
