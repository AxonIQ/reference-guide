# Event Dispatchers

Event publication can from a couple of locations within your Axon Framework application. In general, these can be grouped in two major areas:

1. Dispatching events from an Aggregate, and
2. Dispatching events from regular components 

This page will describe how to get an event message on the event bus from both locations. For more specifics regarding event publication and storage implementations in Axon Framework, read [this](event-dispatchers.md) section.

## Dispatching events from an Aggregate

The [Aggregate](../axon-framework-commands/modeling/aggregate.md) or it's [Entities](../axon-framework-commands/modeling/multi-entity-aggregates.md) are typically the starting point of all event messages. The Event Message simply is the notification that a decision has been made; a successful resolution of handling a command message.

To publish an event from an Aggregate, it is required to do this from the lifecycle of the Aggregate instance. This is mandatory as we want the Aggregate identifier to be tied to the Event message. It is also of the essence that the events originate in order. This is achieved by adding a sequence number to every event from an Aggregate.

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

   The event will first be sent to all the Event Handlers in the Aggregate which are interested.

   This is necessary for [Event Sourcing](../../architecture-overview/event-sourcing.md), to update the Aggregate's state accordingly.

5. After the Aggregate itself has handled the event, it will be published on the `EventBus`.

> **MetaData in Aggregate Event Messages**
>
> The `AggregateLifecycle` also provides an `apply(Object, MetaData)` function. This can be used to attach command-handler specific MetaData.

## Dispatching events from a Non-Aggregate

In the vast majority of cases, the [Aggregates](../axon-framework-commands/modeling/aggregate.md) will publish events by applying them. However, occasionally, it is necessary to publish an event \(possibly from within another component\), directly to the Event Gateway:

```java
private EventGateway eventGateway;

public void dispatchEvent() {
    eventGateway.publish(new CardIssuedEvent("cardId", 100, "shopId"));
}
// omitted class and constructor
```

