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

Although Command Handlers can be placed in regular components (as will be discussed [here](external-command-handler.md), 
it is recommended to define the Command Handlers directly on the Aggregate that contains the state to process this command.

To define a Command Handler in an Aggregate, simply annotate the method which should handle the command with `@CommandHandler`.
The `@CommandHandler` annotated method will become a Command Handler for Command Messages where the _command name_ 
matches fully qualified class name of the first parameter of that method. 
Thus, a method signature of `void handle(RedeemCardCommand cmd)` annotated with `@CommandHandler`, 
will be the Command Handler of the `RedeemCardCommand` Command Messages.

Command Messages can also be [dispatched](dispatching-commands.md) with different _command names_.
To be able to handle those correctly, the `String commandName` value can be specified in the `@CommandHandler` annotation.  

In order for Axon to know which instance of an Aggregate type should handle the Command Message, 
the property carrying the Aggregate Identifier in the command object __must__ be annotated with `@TargetAggregateIdentifier`. 
The annotation may be placed on either the field or an accessor method \(e.g. a getter\) in the Command object.

Taking the `GiftCard` Aggregate as an example, we can identify two Command Handlers on the Aggregate:

```java
import org.axonframework.commandhandling.CommandHandler;
import org.axonframework.modelling.command.AggregateIdentifier;

import static org.axonframework.modelling.command.AggregateLifecycle.apply;

public class GiftCard {

    @AggregateIdentifier
    private String id;
    private int remainingValue;

    @CommandHandler
    public GiftCard(IssueCardCommand cmd) {
        apply(new CardIssuedEvent(cmd.getCardId(), cmd.getAmount()));
    }

    @CommandHandler
    public void handle(RedeemCardCommand cmd) {
        if (cmd.getAmount() <= 0) {
            throw new IllegalArgumentException("amount <= 0");
        }
        if (cmd.getAmount() > remainingValue) {
            throw new IllegalStateException("amount > remaining value");
        }
        apply(new CardRedeemedEvent(id, cmd.getTransactionId(), cmd.getAmount()));
    }
    // omitted event sourcing handlers
}
```

And the Command objects, the `IssueCardCommand` and `RedeemCardCommand` will have the following format:

```java
import org.axonframework.modelling.command.TargetAggregateIdentifier;

public class IssueCardCommand {
    
    @TargetAggregateIdentifier
    private final String cardId;
    private final Integer amount;

    public IssueCardCommand(String cardId, Integer amount) {
        this.cardId = cardId;
        this.amount = amount;
    }
    // omitted getters, equals/hashCode, toString functions
}

public class RedeemCardCommand {

    @TargetAggregateIdentifier
    private final String cardId;
    private final String transactionId;
    private final Integer amount;

    public RedeemCardCommand(String cardId, String transactionId, Integer amount) {
        this.cardId = cardId;
        this.transactionId = transactionId;
        this.amount = amount;
    }
    // omitted getters, equals/hashCode, toString functions
}
```

The `cardId` present in both commands is the reference to a `GiftCard` instance 
and thus is annotated withe the `@TargetAggregateIdentifier` annotation. 
Commands that create an Aggregate instance do not need to identify the target aggregate identifier, 
as there is no Aggregate in existence yet.
It is nonetheless recommended for consistency to annotate the Aggregate Identifier on them as well.

If you prefer to use another mechanism for routing commands, 
the behavior can be overridden by supplying a custom `CommandTargetResolver`. 
This class should return the Aggregate Identifier and expected version \(if any\) based on a given command.

> **Note**
>
> When the `@CommandHandler` annotation is placed on an aggregate's constructor, 
> the respective command will create a new instance of that aggregate and add it to the repository. 
> Those commands do not require to target a specific aggregate instance. 
> Therefore, those commands do not require any `@TargetAggregateIdentifier` or `@TargetAggregateVersion` annotations, 
> nor will a custom `CommandTargetResolver` be invoked for these commands.

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

## Aggregate Lifecycle Operations

There are a couple of operations which are desirable to be performed whilst in the life cycle of an Aggregate.
To that end, the `AggregateLifecycle` class in Axon provides a couple of static functions:

1. `apply(Object)` and `apply(Object, MetaData)`: The `AggregateLifecycle#apply` will publish an Event message on an 
`EventBus` such that it is known to have originated from the Aggregate executing the operation. 
There is the possibility to provide just the Event `Object` or both the Event and some specific [MetaData](../../configuring-infrastructure-components/messaging-concepts/message-anatomy.md#metadata).  
2. `createNew(Class, Callable)`: Instantiate a new Aggregate as a result of handling a Command. 
Read [this](aggregate-creation-from-aggregate.md) for more details on this.
3. `isLive()`: Check to verify whether the Aggregate is in a 'live' state. 
An Aggregate is regarded to be 'live' if it has finished replaying historic events to recreate it's state. 
If the Aggregate is thus in the process of being event sourced, an `AggregateLifecycle.isLive()` call would return `false`.
Using this `isLive()` method, you can perform activity that should only be done when handling newly generated events.  
4. `markDeleted()`: Will mark the Aggregate instance calling the function as being 'deleted'.
Useful if the domain specifies a given Aggregate can be removed/deleted/closed, 
 after which it should no longer be allowed to handle any Commands.
This function should be called from an `@EventSourcingHandler` annotated function to ensure that _being marked deleted_ 
is part of that Aggregate's state.
