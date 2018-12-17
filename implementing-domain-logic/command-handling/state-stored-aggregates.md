# State-stored aggregates

This type of aggregates are storing the current state. It is not possible to rebuild the state of this aggregates based on the events that they have published in the past.

An aggregate root must declare a field that contains the aggregate identifier. This identifier must be initialized at the latest when the first event is published. This identifier field must be annotated by the `@AggregateIdentifier` annotation. If you use JPA and have JPA annotations on the aggregate, Axon can also use the `@Id` annotation provided by JPA.

Aggregates may use the `AggregateLifecycle.apply()` method to register events for publication. Unlike the `EventBus`, where messages need to be wrapped in an `EventMessage`, `apply()` allows you to pass in the payload object directly.

```java
import static org.axonframework.commandhandling.model.AggregateLifecycle.apply;

@Entity // Mark this aggregate as a JPA Entity
public class MyAggregate {

    @Id // When annotating with JPA @Id, the @AggregateIdentifier annotation 
        // is not necessary
    private String id;

    // fields containing state...

    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        // ... update state
        apply(new MyAggregateCreatedEvent(...));
    }

    // constructor needed by JPA
    protected MyAggregate() {
    }
}
```

Entities within an Aggregate can listen to the events the Aggregate publishes, by defining an `@EventHandler` annotated method. These methods will be invoked when an EventMessage is published \(before any external handlers are published\).
