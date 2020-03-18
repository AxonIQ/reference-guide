# Aggregate

This chapter will cover the basics on how to implement an ['Aggregate'](../../../architecture-overview/ddd-cqrs-concepts.md#aggregates). For more details on what an Aggregate is read the [DDD and CQRS concepts](../../../architecture-overview/ddd-cqrs-concepts.md) page.

## Basic Aggregate Structure

An Aggregate is a regular object, which contains state and methods to alter that state. When creating the Aggregate object, you are effectively creating the 'Aggregate Root', typically carrying the name of the entire Aggregate. For the purpose of this description the 'Gift Card' domain will be used, which brings us the `GiftCard` as the Aggregate \(Root\). By default, Axon will configure your Aggregate as an 'Event Sourced' Aggregate \(as described [here](../../../architecture-overview/#event-sourcing)\). Henceforth our basic `GiftCard` Aggregate structure will focus on the Event Sourcing approach:

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
    // omitted command handlers and event sourcing handlers
}
```

There are a couple of noteworthy concepts from the given code snippets, marked with numbered Java comments referring to the following bullets:

1. The `@AggregateIdentifier` is the external reference point to into the `GiftCard` Aggregate. 

   This field is a hard requirement, as without it Axon will not know to which Aggregate a given Command is targeted.

2. A `@CommandHandler` annotated constructor, or differently put the 'command handling constructor'. 

   This annotation tells the framework that the given constructor is capable of handling the `IssueCardCommand`.

   The `@CommandHandler` annotated functions are the place where you would put your _decision-making/business logic_. 

3. The static `AggregateLifecycle#apply(Object...)` is what is used when an Event Message should be published. 

   Upon calling this function the provided `Object`s will be published as `EventMessage`s within the scope of the Aggregate they are applied in.

4. Using the `@EventSourcingHandler` is what tells the framework that the annotated function should be called when the Aggregate is 'sourced from its events'.

   As all the Event Sourcing Handlers combined will form the Aggregate, this is where all the _state changes_ happen.

   Note that the Aggregate Identifier **must** be set in the `@EventSourcingHandler` of the very first Event published by the aggregate. 

   This is usually the creation event. Lastly, `@EventSourcingHandler` annotated functions are resolved using specific rules.

   These rules are the same for the `@EventHandler` annotated methods, and are thoroughly explained in [Annotated Event Handler](../../event-handling/handling-events.md#handling-events).

5. A no-arg constructor, which is required by Axon.

   Axon Framework uses this constructor to create an empty aggregate instance before initializing it using past Events. 

   Failure to provide this constructor will result in an exception when loading the Aggregate.

> **Modifiers for Message Handling functions**
>
> Event Handler methods may be private, as long as the security settings of the JVM allow the Axon Framework to change the accessibility of the method. This allows you to clearly separate the public API of your Aggregate, which exposes the methods that generate events, from the internal logic, which processes the events.
>
> Most IDE's have an option to ignore "unused private method" warnings for methods with a specific annotation. Alternatively, you can add an `@SuppressWarnings("UnusedDeclaration")` annotation to the method to make sure you do not accidentally delete an event handler method.

## .

## Aggregate Lifecycle Operations

There are a couple of operations which are desirable to be performed whilst in the life cycle of an Aggregate. To that end, the `AggregateLifecycle` class in Axon provides a couple of static functions:

1. `apply(Object)` and `apply(Object, MetaData)`: The `AggregateLifecycle#apply` will publish an Event message on an `EventBus` such that it is known to have originated from the Aggregate executing the operation. 

   There is the possibility to provide just the Event `Object` or both the Event and some specific [MetaData](../../messaging-concepts/message-anatomy.md#meta-data).  

2. `createNew(Class, Callable)`: Instantiate a new Aggregate as a result of handling a Command. 

   Read [this](aggregate-creation-from-aggregate.md) for more details on this.

3. `isLive()`: Check to verify whether the Aggregate is in a 'live' state. 

   An Aggregate is regarded to be 'live' if it has finished replaying historic events to recreate it's state. 

   If the Aggregate is thus in the process of being event sourced, an `AggregateLifecycle.isLive()` call would return `false`.

   Using this `isLive()` method, you can perform activity that should only be done when handling newly generated events.  

4. `markDeleted()`: Will mark the Aggregate instance calling the function as being 'deleted'.

   Useful if the domain specifies a given Aggregate can be removed/deleted/closed, after which it should no longer be allowed to handle any Commands.

   This function should be called from an `@EventSourcingHandler` annotated function to ensure that _being marked deleted_ is part of that Aggregate's state.

## Aggregate Command Handler Creation Policy

In the [Handling Commands In An Aggregate](aggregate.md#handling-commands-in-an-aggregate) section we have depicted the `GiftCard` aggregate with roughly two types of command handlers:

1. `@CommandHandler` annotated constructors
2. `@CommandHandler` annotated methods

Option 1 will always expect to be the instantiation of the `GiftCard` aggregate, whilst option 2 expects to be targeted towards an existing aggregate instance. Although this may be the default, there is the option to define a _creation policy_ on a command handler. This can be achieved by adding the `@CreationPolicy` annotation to a command handler annotated method, like so:

```java
import org.axonframework.commandhandling.CommandHandler;
import org.axonframework.modelling.command.CreationPolicy;
import org.axonframework.modelling.command.AggregateCreationPolicy;

public class GiftCard {

    public GiftCard() {
        // Required no-op constructor
    }

    @CommandHandler
    @CreationPolicy(AggregateCreationPolicy.ALWAYS)
    public void handle(IssueCardCommand cmd) {
        // An `IssueCardCommand`-handler which will create a `GiftCard` aggregate 
    }

    @CommandHandler
    @CreationPolicy(AggregateCreationPolicy.CREATE_IF_MISSING)
    public void handle(CreateOrRechargeCardCommand cmd) {
        // A 'CreateOrRechargeCardCommand'-handler which creates a `GiftCard` aggregate if it did not exist
        // Otherwise, it will update an existing `GiftCard` aggregate.
    }
    // omitted aggregate state, command handling logic and event sourcing handlers
}
```

As is shown above, the `@CreationPolicy` annotation requires stating the `AggregateCreationPolicy`. This enumeration has the following options available:

* `ALWAYS` - A creation policy of "always" will expect to instantiate the aggregate. 

  This effectively works like a command handler annotated constructor.

* `CREATE_IF_MISSING` - A creation policy of "create if missing" can either create an aggregate or act on an existing instance.

  This policy should be regarded as a create or update approach of an aggregate.

* `NEVER` - A creation policy of "never" will be handled on an existing aggregate instance.

  This effectively works like any regular command handler annotated method.

