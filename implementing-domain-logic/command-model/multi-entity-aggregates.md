# Multi-entity aggregates

Complex business logic often requires more than what an aggregate with only an aggregate root can provide. In that case, it is important that the complexity is spread over a number of entities within the aggregate. When using event sourcing, not only the aggregate root needs to use events to trigger state transitions, but so does each of the entities within that aggregate.

> **Note** 
>
> A common misinterpretation of the rule that aggregates should not expose state is that none of the entities should contain any property accessor methods. This is not the case. In fact, an aggregate will probably benefit a lot if the entities _within_ the aggregate expose state to the other entities in that same aggregate. However, is is recommended not to expose the state _outside_ of the aggregate.

Axon provides support for event sourcing in complex aggregate structures. Entities are, just like the aggregate root, simple objects. The field that declares the child entity must be annotated with `@AggregateMember`. This annotation tells Axon that the annotated field contains a class that should be inspected for command and event handlers.

When an entity \(including the aggregate root\) applies an event, it is handled by the aggregate root first, and then bubbles down through all `@AggregateMember` annotated fields to its child entities.

> **Note** There is a way to filter the entities which would handle an event applied by the Aggregate Root. This can be achieved by using `eventForwardingMode` attribute of `@AggregateMember` annotation. By default, an event is propagated to **all** child entities. An event can be blocked using `ForwardNone` event forwarding mode \(see listing below\).
>
> ```java
> public class MyAggregate {
>    ...
>    @AggregateMember(eventForwardingMode = ForwardNone.class)
>    private MyEntity myEntity;
>    ...
> }
> ```
>
> If you want to forward an event to the entity only in a case when an event message has matching entity identifier use `ForwardMatchingInstances` event forwarding mode. Entity identifier matching will be done based on specified `routingKey` on `@AggregateMember` annotation. If `routingKey` is not specified on `@AggregateMember` annotation, matching will be done based on `routingKey` attribute on `@EntityId` annotation. If `routingKey` is not specified on `@EntityId` annotation matching will be done based on field name of entity identifier. Let's take a look at example on how to define `ForwardMatchingInstances` event forwarding mode with specifying a routing key for the entity identifier:
>
> ```java
> public class MyAggregate {
>    ...
>    @AggregateMember(eventForwardingMode = ForwardMatchingInstances.class)
>    private MyEntity myEntity;
>    ...
> }
> ...
> public class MyEntity {
>    ...
>    @EntityId(routingKey = "myEntityId")
>    private String id;
>    ...
>    @EventSourcingHandler
>    public void on(MyEvent event) {
>        // handle event
>    }
> }
> ...
> public class MyEvent {
>    ...
>    private String myEntityId;
>    ...
> }
> ```

Fields that \(may\) contain child entities must be annotated with `@AggregateMember`. This annotation may be used on a number of field types:

* the entity type, directly referenced in a field
* inside fields containing an `Iterable` \(which includes all collections, such as `Set`, `List`, etc\)
* inside the values of fields containing a `java.util.Map`
