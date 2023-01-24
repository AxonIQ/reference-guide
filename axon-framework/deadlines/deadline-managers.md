# Deadline Managers

Deadlines can be scheduled from sagas and aggregates. The `DeadlineManager` component is responsible for scheduling deadlines and invoking `@DeadlineHandler`when the deadline is met. The `DeadlineManager` can be injected as a resource. It has three flavors: `SimpleDeadlineManager`, `JonRunrDeadlineManager` and `QuartzDeadlineManager`

## Scheduling a Deadline

A deadline can be scheduled by providing a `Duration` after which it will be triggered \(or an `Instant` at which it will be triggered\) and a _deadline name_.

> **Scheduled Events or Scheduled Deadlines**
>
> Unlike [Event Scheduling](event-schedulers.md), when a deadline is triggered there will be no storing of the published message. Scheduling/Triggering a deadline does not involve an `EventBus` \(or `EventStore`\), hence the message **is not** stored.

```java
class DeadlineSchedulingComponent {
    void scheduleMyDeadline() {
        String deadlineId = 
            deadlineManager.schedule(Duration.ofMillis(500), "myDeadline");
        // For example store the `deadlineId`
    }
}
```

As a result we receive a `deadlineId` which can be used to cancel the deadline. In most cases, storing this `deadlineId` as a field within your Aggregate/Saga is the most convenient. Cancelling a deadline could come in handy when a certain event means that the previously scheduled deadline has become obsolete \(e.g. there is a deadline for paying the invoice, but the client paid the amount which means that the deadline is obsolete and can be canceled\).

```java
class DeadlineCancelingComponent {
    void cancelMyDeadline(String deadlineId) {
        deadlineManager.cancelSchedule("myDeadline", deadlineId);
    }
}
```

Note that there are more options to cancel a deadline next to the previously mentioned:

* `cancelAll(String deadlineName)`

  Cancels _every_ scheduled deadline matching the given `deadlineName`.

  Note that this thus also cancels deadlines from other aggregate and/or saga instances matching the name.

* `cancelAllWithinScope(String deadlineName)`

  Cancels a scheduled deadline matching the given `deadlineName`, _within_ the `Scope` the method is invoked in.

  For example, if this operation is performed from within "aggregate instance X",

  the `ScopeDescriptor` from "aggregate instance X" will be used to cancel.

* `cancelAllWithinScope(String deadlineName, ScopeDescriptor scope)`

  Cancels a scheduled deadline matching the given `deadlineName` _and_ `ScopeDescriptor`.

  This allows canceling a deadline by name from differing scopes then the one it's executed in.

> **Caveats for the JobRunr implementation.**
> 
> Since JobRunr has no way to search for deadlines, besides by id, all of the `cancelAll` methods are not implemented for the `JobRunrDeadlineManager`.
> This might change in the future, but only in combination with using the [Pro version](https://github.com/AxonFramework/AxonFramework/issues/2507).

If you need contextual data about the deadline when the deadline is being handled, you can attach a deadline payload when scheduling a deadline:

```java
class DeadlineSchedulingWithPayloadComponent {
    void scheduleMyDeadlineWithPayload() {
        String deadlineId = deadlineManager.schedule(
            Duration.ofMillis(500), "myDeadline", 
            new MyDeadlinePayload(/* some user specific parameters */)
        );
        // For example store the `deadlineId`
    }
}
```

## Handling a Deadline

We have now seen how to schedule a deadline. When the scheduled time is met, the corresponding `@DeadlineHandler` is invoked. A `@DeadlineHandler` is a message handler like any other in Axon - it is possible to inject parameters for which `ParameterResolver`s exist.

> **The Scope of a Deadline**
>
> When scheduling a deadline, the context from where it was scheduled is taken into account. This means a scheduled deadline will only be triggered in its originating context. Thus, any `@DeadlineHandler` annotated function you wish to be called on a met deadline, must be in the same Aggregate/Saga from which it was scheduled.
>
> Axon calls this context a `Scope`. If necessary, implementing and providing your own `Scope` will allow you to schedule deadlines in your custom, 'scoped' components.
> 
> A Saga can end its lifecycle when `@EndSaga` is added on a deadline handler. 

A `@DeadlineHandler` is matched based on the deadline name and the deadline payload.

```java
@DeadlineHandler(deadlineName = "myDeadline")
public void on(MyDeadlinePayload deadlinePayload) {
    // handle the deadline
}
```

If the deadline's name is not defined in the `@DeadlineHandler`, matching will proceed based on the deadline payload alone.

```java
@DeadlineHandler
public void on(MyDeadlinePayload deadlinePayload) {
    // handle the deadline
}
```

If we scheduled a deadline without a specific payload, the `@DeadlineHandler` does not have to specify the payload.

```java
@DeadlineHandler(deadlineName = "payloadlessDeadline")
public void on() {
    // handle the deadline
}
```

## Using Time In Your  Application

In cases where applications need to access the clock, they can take advantage of the clock used in the EventMessage, by accessing `GenericEventMessage.clock`. This clock is set to Clock.systemUTC at runtime, and manipulated to simulate time during [testing](../testing/).

```java
public void handle(PublishTime cmd) {
    apply(new TimePublishedEvent(GenericEventMessage.clock.instant()));
}
```

Note that the current timestamp is automatically added to the EventMessage. If handlers only need to rely on the timestamp the event was published, they can access that timestamp directly, as described in [Handling Events](../events/event-handlers.md).

## Configuration

Spring Boot users will need to define a `DeadlineManager` bean using one of the available implementations. 

Spring Boot users who want to use the JobRunr deadline managers can add [`jobrunr-spring-boot-starter`](https://mvnrepository.com/artifact/org.jobrunr/jobrunr-spring-boot-starter) as a dependency.
The needed bean configuration should look like this:

```java
@Bean
public DeadlineManager deadlineManager(
        @Qualifier("eventSerializer") final Serializer serializer,
        final JobScheduler jobScheduler,
        final ScopeAwareProvider scopeAwareProvider,
        final TransactionManager transactionManager,
        final Spanfactory spanfactory
) {
    return JobRunrDeadlineManager.builder()
            .jobScheduler(jobScheduler)
            .scopeAwareProvider(scopeAwareProvider)
            .serializer(serializer)
            .transactionManager(transactionManager)
            .spanFactory(spanfactory)
            .build();
}
