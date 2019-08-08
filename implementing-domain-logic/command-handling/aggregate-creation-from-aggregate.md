# Aggregate creation from another Aggregate

Regularly instantiating a new Aggregate is done by issuing a creation command which is handled by a `@CommandHandler`
 annotated Aggregate constructor.
Such commands could for example be published by a simple [REST endpoint](../connecting-the-ui/command-publishing-use-cases.md)
 or an [Event Handling Component](../event-handling/handling-events.md) as a reaction to a certain event. 
Sometimes the Domain however describes certain Entities to be created from another Entity.
In this scenario it would thus be more faithful to the domain to instantiate an Aggregate from it's parent Aggregate.

## How to create an Aggregate from another Aggregate
To achieve this...

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

\(1\) The first parameter of the `AggregateLifecycle#createNew()` method is the type of Aggregate to be created. 
The second parameter is the factory method - the method to be used in order to instantiate the desired Aggregate.

> **Note** 
> 
> Creation of a new Aggregate should be done in a Command Handling function rather than in an Event Handling function
>  \(given the usage of Event Sourced Aggregate\). 
> Rationale: we do not want to create new Aggregates when we are sourcing a given Aggregate -
>  previously created aggregate will be Event Sourced based on its events. 
> However, if you try to create a new Aggregate while Axon is replaying events, an `UnsupportedOperationException` will be thrown.

<!--
## Aggregate-from-Aggregate Use Cases
-->