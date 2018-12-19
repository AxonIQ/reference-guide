# Aggregate

An aggregate is always accessed through a single entity, called the Aggregate Root. Usually, the name of this entity is the same as that of the aggregate entirely. For example, an Order aggregate may consist of an Order entity, which references several OrderLine entities. Order and Orderline together, form the aggregate.

An aggregate is a regular object, which contains state and methods to alter that state. Although, not entirely correct according to CQRS principles, it is also possible to expose the state of the aggregate through accessor methods.

## Event sourced aggregates

It is common for CQRS systems to rebuild the state of an aggregate based on the events that it has published in the past. For this to work, all state changes must be represented by an event.

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

## Handling commands in an Aggregate

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

### Returning results from command handlers

In some cases, the component dispatching a command needs information about the processing results of a command. A command handler method can return a value from its method. That value will be provided to the sender as the result of the command.

One exception is the `@CommandHandler` on an aggregate's constructor. In this case, instead of returning the return value of the method \(which is the Aggregate itself\), the value of the `@AggregateIdentifier` annotated field is returned instead

> **Note**
>
> While it is possible to return results from commands, it should be used sparsely. The intent of the command should never be in getting a value, as that would be an indication the message should be designed as a [Query Message ](https://docs.axoniq.io/reference-guide/1.2-domain-logic/query-handling)instead. A typical example for a Command result is the identifier of a newly created entity.