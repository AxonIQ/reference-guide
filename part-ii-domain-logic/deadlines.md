# Deadlines

The 'Deadline concept' in the Axon Framework is a mechanism which enables certain actions (in our case a `@DeadlineHandler` annotated method) to be executed after a certain amount of time. The context of this execution is an Aggregate or a Saga in which the Deadline was scheduled. If the Deadline becomes obsolete there is the possibility to cancel it as well.  

Deadlines can be scheduled from Sagas and Aggregates. The `DeadlineManager` component is responsible for scheduling deadlines and invoking `@DeadlineHandler`s when the deadline is met. The `DeadlineManager` can be injected as a resource. It has two flavors: `SimpleDeadlineManager` and `QuartzDeadlineManager`, just like the [Event Scheduling](sagas.md#keeping-track-of-deadlines) mechanism for Sagas. 

## Scheduling a Deadline

A deadline can be scheduled by providing a `Duration` after which it will be triggered (or `Instance` at which it will be triggered) and a name.

> **Note**
>  
> Unlike [Event Scheduling](sagas.md#keeping-track-of-deadlines), when a Deadline is triggered there will be no storing of the published Message. Scheduling/Triggering a deadline does not involve an EventBus (or EventStore), hence the Message is not stored.

```java
String deadlineId = deadlineManager.schedule(Duration.ofMillis(500), "myDeadline");
```

As a result we receive a `deadlineId` which can be used to cancel the deadline. In most cases, storing this `deadlineId` as a field within your Aggregate/Saga is the most convenient. Cancelling a deadline could for example come in handy when a certain event means that the previously scheduled deadline has become obsolete (e.g. there is a deadline for paying the invoice, but the client payed the amount which means that the deadline is obsolete and can be canceled).

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

Lastly, you could also provide the Deadline Identifier to the `DeadlineManager` instead of letting the `DeadlineManager` generate an identifier automatically.

## Handling a Deadline

We have now seen how to schedule a Deadline. When the scheduled time is met, the corresponding `@DeadlineHandler` is invoked. A `@DeadlineHandler` is a Message Handler as any other in Axon - it is possible to inject parameters for which the `ParameterResolver`s exist. 

> **Note** 
>
> When scheduling a deadline, the context from where it was scheduled is taken into account. 
> That means a given scheduled deadline will only be triggered in its originating context. 
> Thus any `@DeadlineHandler` annotated function you wish to be called on a met deadline, must be in the same Aggregate/Saga from which is was scheduled.
>
> Axon calls this Context a Scope. If necessary, implementing and providing your own Scope will allow you to schedule Deadlines in your custom, scoped components.

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