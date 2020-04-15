# Aggregate Creation from another Aggregate

Regularly, instantiating a new Aggregate is done by issuing a creation command which is handled by a `@CommandHandler` annotated Aggregate constructor. Such commands could for example be published by a simple [REST endpoint](https://github.com/domaincomponents/reference-guide/tree/7ae838faa2d2d8045603b108c1e042f7452f59dc/implementing-domain-logic/connecting-the-ui/command-publishing-use-cases.md) or an [Event Handling Component ](../../events/event-handlers.md)as a reaction to a certain event. Sometimes the Domain however describes certain Entities to be created from another Entity. In this scenario it would thus be more faithful to the domain to instantiate an Aggregate from it's parent Aggregate.

> **Aggregate-from-Aggregate Use Case**
>
> The most suitable scenario to create a "child" Aggregate from a "parent" Aggregate, is when the decision to create the child lies within the context of a parent Aggregate. This can for example manifest itself if the parent Aggregate contains the necessary state which can drive this child-creation decision.

## How to create an Aggregate from another Aggregate

Let us assume we have a `ParentAggregate`, that upon handling a certain command will decide to create an `ChildAggregate`. To achieve this, `ParentAggregate` would look something like this:

```java
import org.axonframework.commandhandling.CommandHandler;

import static org.axonframework.modelling.command.AggregateLifecycle.createNew;

public class ParentAggregate {

    @CommandHandler
    public void handle(SomeParentCommand command) {
        createNew(
            ChildAggregate.class, 
            () -> new ChildAggregate(/* provide required constructor parameters if applicable */)
        ); 
    }
    // omitted no-op constructor, event sourcing handlers and other command handlers
}
```

The `AggregateLifecycle#createNew(Class<T>, Callable<T>)` is key to instantiation another Aggregate like our `ChildAggregate` as a reaction to handling a command. The first parameter to the `createNew` method is the `Class` of the Aggregate to be created. The second parameter is the factory method, which expects the outcome to be an object identical to the given type.

The `ChildAggregate` implementation would in this scenario resemble the following format:

```java
import static org.axonframework.modelling.command.AggregateLifecycle.apply;

public class ChildAggregate {

    public ChildAggregate(String aggregateId) {
        apply(new ChildAggregateCreatedEvent(aggregateId));
    }
    // omitted no-op constructor, command and event sourcing handlers
}
```

Note that a `ChildAggregateCreatedEvent` is explicitly applied to notify the `ChildAggregate` was created, as this knowledge would otherwise be enclosed in the `SomeParentCommand` command handler of the `ParentAggregate`.

> **Creating Aggregates from Event Sourcing Handlers?**
>
> Creation of a new Aggregate should be done in a command handler rather than in an event sourcing handler. The rationale behind this is that you do not want to create new child Aggregates when a parent Aggregate is sourced from its events, as this would undesirably create new child Aggregate instances
>
> If the `createNew` method is however accidentally called within an event sourcing handler, an `UnsupportedOperationException` will be thrown as stop gap solution.

