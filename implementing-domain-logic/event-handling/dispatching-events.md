# Dispatching Events

Event publication from a couple of locations when your Axon Framework application.
In general, these can be grouped in two major areas:

1. Dispatching events from an Aggregate, and
2. Dispatching events from regular components 

This page will describe how to get an event message on the event bus from both locations.

## Dispatching events from an Aggregate

The [Aggregate](../command-handling/aggregate.md) or it's [Entities](../command-handling/multi-entity-aggregates.md) 
 are typically the starting point of all event messages.
The Event Message simply is the notification that a decision has been made;
 a successful resolution of handling a command message.

To publish an event from an Aggregate, it is required to do this from the lifecycle of the Aggregate instance.
This is mandatory as we want the Aggregate identifier to be tied to the Event message.
It is also of the essence that the events originate in order.
This is achieved by adding a sequence number to every event from an Aggregate.

The `AggregateLifecycle` provides a simple means to achieve the above:

```java
import static org.axonframework.modelling.command.AggregateLifecycle.apply;

public class GiftCard {
    
    @CommandHandler
    public GiftCard(IssueCardCommand cmd) {
        AggregateLifecycle.apply(new CardIssuedEvent(cmd.getCardId(), cmd.getAmount()));
    }
    // omitted state, command and event sourcing handlers
}
```

The `AggregateLifecycle#apply(Object)` will go through a number of steps:

1. The current scope of the Aggregate is retrieved.
2. The last known sequence number of the Aggregate is used to set the sequence number of the event to publish.
3. The provided Event payload, the `Object`, will be wrapped in an `EventMessage`.
The `EventMessage` will also receive the `sequenceNumber` from the previous step, as well as the Aggregate it's identifier.
4. The Event Message will be published from here on. 
The event will first be send to all the Event Handlers in the Aggregate which are interested.
This is necessary for [Event Sourcing](../command-handling/aggregate.md#basic-aggregate-structure),
 to update the Aggregate's state accordingly.
5. After the Aggregate itself has handled the event, it will be published on the `EventBus`. 

> **MetaData in Aggregate Event Messages**
>
> The `AggregateLifecycle` also provides a `apply(Object, MetaData)` function.
> This can be used to attach command-handler specific [MetaData](../../configuring-infrastructure-components/messaging-concepts/message-anatomy.md#metadata).

## Dispatching events from a Non-Aggregate

In the vast majority of cases, the [Aggregates](#dispatching-events-from-an-aggregate) will publish events by applying them. 
However, occasionally, it is necessary to publish an event (possibly from within another component),
 directly to the Event Bus:

```java
private EventBus eventBus; // 1.

public void dispatchEvent() {
    // 2. & 3.
    eventBus.publish(GenericEventMessage.asEventMessage(new CardIssuedEvent("cardId", 100, "shopId")));
}
// omitted class and constructor 
```

1. The `EventBus` interface which allows the publication of events.
2. The `GenericEventMessage#asEventMessage(Object)` method allows you to wrap any object into an `EventMessage`. 
If the passed object is already an EventMessage, it is simply returned.
3. `publish(EventMessage...)` is used to actually publish an event.
As shown, the event payload, the `CardIssuedEvent`, should be wrapped in an `EventMessage`. 
