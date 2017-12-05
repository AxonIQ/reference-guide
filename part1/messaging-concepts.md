Messaging concepts
==================

One of the core concepts in Axon is messaging. All communication between components is done using message objects. This gives these components the location transparency needed to be able to scale and distribute these components when necessary.

Although all these messages implement the `Message` interface, there is a clear distinction between the different types of Messages and how they are treated.

All Messages contain a payload, meta data and unique identifier. The payload of the message is the functional description of what the message means. The combination of the class name of this object and the data it carries, describe the application's meaning of the message. The meta data allows you to describe the context in which a message is being sent. You can, for example, store tracing information, to allow the origin or cause of messages to be tracked. You can also store information to describe the security context under which a command is being executed.

> **Note**
>
> Note that all Messages are immutable. Storing data in a Message actually means creating a new Message based on the previous one, with extra information added to it. This guarantees that Messages are safe to use in a multi-threaded and distributed environment.

Commands
--------

Commands describe an intent to change the application's state. They are implemented as (preferably read-only) POJOs that are wrapped using one of the `CommandMessage` implementations.

Commands always have exactly one destination. While the sender doesn't care which component handles the command or where that component resides, it may be interesting in knowing the outcome of it. That's why Command messages sent over the Command Bus allow for a result to be returned.

Events
------

Events are objects that describe something that has occurred in the application. A typical source of Events is the Aggregate. When something important has occurred within the Aggregate, it will raise an Event. In Axon Framework, Events can be any object. You are highly encouraged to make sure all Events are serializable.

When Events are dispatched, Axon wraps them in an `EventMessage`. The actual type of Message used depends on the origin of the Event. When an Event is raised by an Aggregate, it is wrapped in a `DomainEventMessage` (which extends `EventMessage`). All other Events are wrapped in an `EventMessage.` Aside from common `Message` attributes like a unique Identifier an `EventMessage` also contains a timestamp. The `DomainEventMessage` additionally contains the type and identifier of the aggregate that raised the Event. It also contains the sequence number of the event in the aggregate's event stream, which allows the order of events to be reproduced.

> **Note**
>
> Even though the `DomainEventMessage` contains a reference to the Aggregate Identifier, you should always include the identifier in the actual Event itself as well. The identifier in the DomainEventMessage is used by the EventStore to store events and may not always provide a reliable value for other purposes.

The original Event object is stored as the Payload of an EventMessage. Next to the payload, you can store information in the Meta Data of an Event Message. The intent of the Meta Data is to store additional information about an Event that is not primarily intended as business information. Auditing information is a typical example. It allows you to see under which circumstances an Event was raised, such as the User Account that triggered the processing, or the name of the machine that processed the Event.

> **Note**
>
> In general, you should not base business decisions on information in the meta-data of event messages. If that is the case, you might have information attached that should really be part of the Event itself instead. Meta-data is typically used for reporting, auditing and tracing.

Although not enforced, it is good practice to make domain events immutable, preferably by making all fields final and by initializing the event within the constructor. Consider using a Builder pattern if Event construction is too cumbersome.

> **Note**
>
> Although Domain Events technically indicate a state change, you should try to capture the intention of the state in the event, too. A good practice is to use an abstract implementation of a domain event to capture the fact that certain state has changed, and use a concrete sub-implementation of that abstract class that indicates the intention of the change. For example, you could have an abstract `AddressChangedEvent`, and two implementations `ContactMovedEvent` and `AddressCorrectedEvent` that capture the intent of the state change. Some listeners don't care about the intent (e.g. database updating event listeners). These will listen to the abstract type. Other listeners do care about the intent and these will listen to the concrete subtypes (e.g. to send an address change confirmation email to the customer).
>
> ![ "Adding intent to events](state-change-intent.png)

When dispatching an Event on the Event Bus, you will need to wrap it in an Event Message. The `GenericEventMessage` is an implementation that allows you to wrap your Event in a Message. You can use the constructor, or the static `asEventMessage()` method. The latter checks whether the given parameter doesn't already implement the `Message` interface. If so, it is either returned directly (if it implements `EventMessage`,) or it returns a new `GenericEventMessage` using the given `Message`'s payload and Meta Data. If an Event is applied (published) by an Aggregate Axon will automatically wrap the Event in a `DomainEventMessage` containing the Aggregate's Identifier, Type and Sequence Number. 

Queries
-------

Queries describe a request for information or state. A query can have multiple handlers. When dispatching queries, the client indicates whether he wants a result from one or from all available query handlers.

Unit of Work
------------

The Unit of Work is an important concept in the Axon Framework, though in most cases you are unlikely to interact with it directly. The processing of a message is seen as a single unit. The purpose of the Unit of Work is to coordinate actions performed during the processing of a message (Command, Event or Query). Components can register actions to be performed during each of the stages of a Unit of Work, such as onPrepareCommit or onCleanup.

You are unlikely to need direct access to the Unit of Work. It is mainly used by the building blocks that Axon provides. If you do need access to it, for whatever reason, there are a few ways to obtain it. The Handler receives the Unit Of Work through a parameter in the handle method. If you use annotation support, you may add a parameter of type `UnitOfWork` to your annotated method. In other locations, you can retrieve the Unit of Work bound to the current thread by calling `CurrentUnitOfWork.get()`. Note that this method will throw an exception if there is no Unit of Work bound to the current thread. Use `CurrentUnitOfWork.isStarted()` to find out if one is available.

One reason to require access to the current Unit of Work is to attach resources that need to be reused several times during the course of message processing, or if created resources need to be cleaned up when the Unit of Work completes. In such case, the `unitOfWork.getOrComputeResource()` and the lifecycle callback methods, such as `onRollback()`, `afterCommit()` and `onCleanup()` allow you to register resources and declare actions to be taken during the processing of this Unit of Work.

> **Note**
>
> Note that the Unit of Work is merely a buffer of changes, not a replacement for Transactions. Although all staged changes are only committed when the Unit of Work is committed, its commit is not atomic. That means that when a commit fails, some changes might have been persisted, while others are not. Best practices dictate that a Command should never contain more than one action. If you stick to that practice, a Unit of Work will contain a single action, making it safe to use as-is. If you have more actions in your Unit of Work, then you could consider attaching a transaction to the Unit of Work's commit. Use `unitOfWork.onCommit(..)` to register actions that need to be taken when the Unit of Work is being committed.

Your handlers may throw an Exception as a result of processing a message. By default, unchecked exceptions will cause the UnitOfWork to roll back all changes. As a result, scheduled side effects are cancelled.

Axon provides a few Rollback strategies out-of-the-box:
 - `RollbackConfigurationType.NEVER`, will always commit the Unit of Work,
 - `RollbackConfigurationType.ANY_THROWABLE`, will always roll back when an exception occurs,
 - `RollbackConfigurationType.UNCHECKED_EXCEPTIONS`, will roll back on Errors and Runtime Exception
 - `RollbackConfigurationType.RUNTIME_EXCEPTION`, will roll back on Runtime Exceptions (but not on Errors)

When using Axon components to process messages, the lifecycle of the Unit of Work will be automatically managed for you. If you choose not to use these components, but implement processing yourself, you will need to programmatically start and commit (or roll back) a Unit of Work instead.

In most cases, the `DefaultUnitOfWork` will provide you with the functionality you need. It expects processing to happen within a single thread. To execute a task in the context of a Unit Of Work, simply call `UnitOfWork.execute(Runnable)` or `UnitOfWork.executeWithResult(Callable)` on a new `DefaultUnitOfWork`. The Unit Of Work will be started and committed when the task completes, or rolled back if the task fails. You can also choose to manually start, commit or rollback the Unit Of Work if you need more control.

Typical usage is as follows:

``` java
UnitOfWork uow = DefaultUnitOfWork.startAndGet(message);
// then, either use the autocommit approach:
uow.executeWithResult(() -> ... logic here);

// or manually commit or rollback:
try {
    // business logic comes here
    uow.commit();
} catch (Exception e) {
    uow.rollback(e);
    // maybe rethrow...
}
```

A Unit of Work knows several phases. Each time it progresses to another phase, the UnitOfWork Listeners are notified.

-   Active phase: this is where the Unit of Work is started. The Unit of Work is generally registered with the current thread in this phase (through `CurrentUnitOfWork.set(UnitOfWork)`). Subsequently the message is typically handled by a message handler in this phase.

-   Commit phase: after processing of the message is done but before the Unit of Work is committed, the `onPrepareCommit` listeners are invoked. If a Unit of Work is bound to a transaction, the `onCommit` listeners are invoked to commit any supporting transactions. When the commit succeeds, the `afterCommit` listeners are invoked. If a commit or any step before fails, the `onRollback` listeners are invoked. The message handler result is contained in the `ExecutionResult` of the Unit Of Work, if available.

-   Cleanup phase: This is the phase where any of the resources held by this Unit of Work (such as locks) are to be released. If multiple Units Of Work are nested, the cleanup phase is postponed until the outer unit of work is ready to clean up.

The message handling process can be considered an atomic procedure; it should either be processed entirely, or not at all. Axon Framework uses the Unit Of Work to track actions performed by the message handlers. After the handler completed, Axon will try to commit the actions registered with the Unit Of Work.

It is possible to bind a transaction to a Unit of Work. Many components, such as the CommandBus and QueryBus implementations and all asynchronously processing Event Processors, allow you to configure a Transaction Manager. This Transaction Manager will then be used to create the transactions to bind to the Unit of Work that is used to manage the process of a Message.

When application components need resources at different stages of message processing, such as a Database Connection or an EntityManager, these resources can be attached to the Unit of Work. The `unitOfWork.getResources()` method allows you to access the resources attached to the current Unit of Work. Several helper methods are available on the Unit of Work directly, to make working with resources easier.

When nested Units of Work need to be able to access a resource, it is recommended to register it on the root Unit of Work, which can be accessed using `unitOfWork.root()`. If a Unit of Work is the root, it will simply return itself.
