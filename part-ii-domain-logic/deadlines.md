# Deadlines

This feature is introduced in 3.3 version of Axon. Until the introduction pf deadlines, it was possible to schedule an event to for example be handled in a Saga. If you require more fine grained control when a deadline is met and for whom, this feature enables that for you. 

Deadlines are possible to be scheduled from Sagas and Aggregates. The `DeadlineManager` component is responsible for scheduling deadlines and invoking `@DeadlineHandler`s when the deadline is met. The `DeadlineManager` can be injected as a resource. It has two flavors: `SimpleDeadlineManager` and `QuartzDeadlineManager`, just like  the [Event Scheduling](sagas.md#keeping-track-of-deadlines) mechanism for Sagas. 

## Scheduling a Deadline

A deadline can be scheduled only by providing a `Duration` after which it will be triggered (or `Instance` at which it will be triggered) and a name.

```java
String deadlineId = deadlineManager.schedule(Duration.ofMillis(500), "myDeadline");
```

As a result we receive a `deadlineId` which can be used to cancel the deadline:

```java
deadlineManager.cancelSchedule("myDeadline", deadlineId);
```

> **Note**
>
> It is possible to cancel all deadlines of a given name by invoking `deadlineManager.cancelAll("myDeadline")`.

If you need some contextual data about the Deadline during the Deadline Handling, you can attach a Deadline Payload when scheduling a Deadline:

```java
String deadlineId = deadlineManager.schedule(Duration.ofMillis(500), "myDeadline", new MyDeadlinePayload(...));
```

Lastly, you could also provide the Deadline Identifier to the `DeadlineManager` instead of letting it generate automatically.

## Handling a Deadline

We have now seen how to schedule a Deadline. When the scheduled time is met, the corresponding `@DeadlineHandler` is invoked. 

> **Note** 
>
> When scheduling a deadline, the context from where it was scheduled is taking into account. 
> That means a given scheduled deadline will only trigger in its originating context. 
> Thus any `@DeadlineHandler` annotated function you wish to be called on a met deadline, must be in the same Aggregate/Saga from which is was scheduled.

A `@DeadlineHandler` is matched based on the Deadline Name and the Deadline Payload. 

```java
@DeadlineHandler(deadlineName = "myDeadline")
public void on(MyDeadlinePayload deadlinePayload) {
    // handle the Deadline
}
```

If the Deadline Name is not defined in the `@DeadlineHandler`, matching will proceed based on the Deadline Payload alone. 

```java
@DeadlineHandler
public void on(MyDeadlinePayload deadlinePayload) {
    // handle the Deadline
}
```

If we scheduled a Deadline without any specific payload, the `@DeadlineHandler` does not have to specify the payload neither. 

```java
@DeadlineHandler(deadlineName = "payloadlessDeadline")
public void on() {
    // handle the Deadline
}
```