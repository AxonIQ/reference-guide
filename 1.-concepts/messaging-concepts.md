# 1.2 Messaging concepts

One of the core concepts in Axon is messaging. All communication between components is done using message objects. This gives these components the location transparency needed to be able to scale and distribute these components when necessary.

Although all these messages implement the `Message` interface, there is a clear distinction between the different types of messages and how they are treated.

All messages contain a payload, meta data and unique identifier. The payload of the message is the functional description of what the message means. The combination of the class name of this object and the data it carries, describe the application's meaning of the message. The metadata allows you to describe the context in which a message is being sent. You can, for example, store tracing information, to allow the origin or cause of messages to be tracked. You can also store information to describe the security context under which a command is being executed.

> **Note**
>
> Note that all messages are immutable. Storing data in a message actually means creating a new message based on the previous one, with extra information added to it. This guarantees that messages are safe to use in a multi-threaded and distributed environment.

## Commands

Commands describe an intent to change the application's state. They are implemented as \(preferably read-only\) POJOs that are wrapped using one of the `CommandMessage` implementations.

Commands always have exactly one destination. While the sender does not care which component handles the command or where that component resides, it may be interesting knowing the outcome of it. That is why command messages sent over the command bus allow for a result to be returned.

## Events

Events are objects that describe something that has occurred in the application. A typical source of events is the aggregate. When something important has occurred within the aggregate, it will raise an event. In Axon Framework, events can be any object. You are highly encouraged to make sure all events are serializable.

When Events are dispatched, Axon wraps them in an `EventMessage`. The actual type of Message used depends on the origin of the event. When an wvent is raised by an aggregate, it is wrapped in a `DomainEventMessage` \(which extends `EventMessage`\). All other events are wrapped in an `EventMessage` . Aside from common `Message` attributes like the unique Identifier an `EventMessage` also contains a timestamp. The `DomainEventMessage` additionally contains the type and identifier of the aggregate that raised the event. It also contains the sequence number of the event in the aggregate's event stream, which allows the order of events to be reproduced.

> **Note**
>
> Even though the `DomainEventMessage` contains a reference to the Aggregate Identifier, you should always include the identifier in the actual Event itself as well. The identifier in the DomainEventMessage is used by the `EventStore` to store events and may not always provide a reliable value for other purposes.

The original event object is stored as the payload of an `EventMessage`. Next to the payload, you can store information in the metadata of an event message. The intent of the metadata is to store additional information about an event that is not primarily intended as business information. Auditing information is a typical example. It allows you to see under which circumstances an Event was raised. Such as the user account that triggered the processing, or the name of the machine that processed the event.

> **Note**
>
> In general, you should not base business decisions on information in the metadata of event messages. If that is the case, you might have information attached that should really be part of the event itself instead. Metadata is typically used for reporting, auditing and tracing.

Although not enforced, it is good practice to make domain events immutable, preferably by making all fields final and by initializing the event within the constructor. Consider using a Builder pattern if event construction is too cumbersome.

> **Note**
>
> Although domain events technically indicate a state change, you should try to capture the intention of the state in the event, too. A good practice is to use an abstract implementation of a domain event to capture the fact that certain state has changed, and use a concrete sub-implementation of that abstract class that indicates the intention of the change. For example, you could have an abstract `AddressChangedEvent`, and two implementations `ContactMovedEvent` and `AddressCorrectedEvent` that capture the intent of the state change. Some listeners don't care about the intent \(e.g. database updating event listeners\). These will listen to the abstract type. Other listeners do care about the intent and these will listen to the concrete subtypes \(e.g. to send an address change confirmation email to the customer\).
>
> ![ &quot;Adding intent to events](../.gitbook/assets/state-change-intent.png)

When dispatching an Event on the Event Bus, you will need to wrap it in an Event Message. The `GenericEventMessage` is an implementation that allows you to wrap your Event in a Message. You can use the constructor, or the static `asEventMessage()` method. The latter checks whether the given parameter doesn't already implement the `Message` interface. If so, it is either returned directly \(if it implements `EventMessage`,\) or it returns a new `GenericEventMessage` using the given `Message`'s payload and Meta Data. If an Event is applied \(published\) by an Aggregate Axon will automatically wrap the Event in a `DomainEventMessage` containing the Aggregate's Identifier, Type and Sequence Number.

## Queries

Queries describe a request for information or state. A query can have multiple handlers. When dispatching queries, the client indicates whether he wants a result from one or from all available query handlers.

## Unit of Work

The `UnitOfWork` is an important concept in the Axon Framework, though in most cases you are unlikely to interact with it directly. The processing of a message is seen as a single unit. The purpose of the unit of work is to coordinate actions performed during the processing of a message \(command, event or query\). Components can register actions to be performed during each of the stages of a `UnitOfWork`, such as `onPrepareCommit` or `onCleanup`.

You are unlikely to need direct access to the `UnitOfWork`. It is mainly used by the building blocks that the Axon Framework provides. If you do need access to it, for whatever reason, there are a few ways to obtain it. The handler receives the unit of work through a parameter in the handle method. If you use annotation support, you may add a parameter of type `UnitOfWork` to your annotated method. In other locations, you can retrieve the unit of work bound to the current thread by calling `CurrentUnitOfWork.get()`. Note that this method will throw an exception if there is no `UnitOfWork` bound to the current thread. Use `CurrentUnitOfWork.isStarted()` to find out if one is available.

One reason to require access to the current unit of work is to attach resources that need to be reused several times during the course of message processing, or if created resources need to be cleaned up when the unit of work completes. In such case, the `unitOfWork.getOrComputeResource()` and the lifecycle callback methods, such as `onRollback()`, `afterCommit()` and `onCleanup()` allow you to register resources and declare actions to be taken during the processing of this unit of work.

> **Note**
>
> Note that the Unit of Work is merely a buffer of changes, not a replacement for transactions. Although all staged changes are only committed when the Unit of Work is committed, its commit is not atomic. That means that when a commit fails, some changes might have been persisted, while others are not. Best practices dictate that a command should never contain more than one action. If you stick to that practice, a unit of work will contain a single action, making it safe to use as-is. If you have more actions in your unit of work, then you could consider attaching a transaction to the unit of work's commit. Use `unitOfWork.onCommit(..)` to register actions that need to be taken when the unit of work is being committed.

Your handlers may throw an Exception as a result of processing a message. By default, unchecked exceptions will cause the `UnitOfWork` to roll back all changes. As a result, scheduled side effects are cancelled.

Axon Framework provides a few rollback strategies out-of-the-box:

* `RollbackConfigurationType.NEVER` - will always commit the `UnitOfWork`
* `RollbackConfigurationType.ANY_THROWABLE`- will always roll back when an exception occurs
* `RollbackConfigurationType.UNCHECKED_EXCEPTIONS`- will roll back on errors and runtime exceptions
* `RollbackConfigurationType.RUNTIME_EXCEPTION`- will roll back on runtime exceptions \(but not on errors\)

When using framework components to process messages, the lifecycle of the unit of work will be automatically managed for you. If you choose not to use these components, but implement processing yourself, you will need to programmatically start and commit \(or roll back\) a unit of work instead.

In most cases, the `DefaultUnitOfWork` will provide you with the functionality you need. It expects processing to happen within a single thread. To execute a task in the context of a unit of work, simply call `UnitOfWork.execute(Runnable)` or `UnitOfWork.executeWithResult(Callable)` on a new `DefaultUnitOfWork`. The unit of work will be started and committed when the task completes, or rolled back if the task fails. You can also choose to manually start, commit or rollback the unit of work if you need more control.

Typical usage is as follows:

```java
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

A `UnitOfWork` knows several phases. Each time it progresses to another phase, the listeners are notified.

* Active phase - this is where the Unit of Work is started. The unit of work is generally registered with the current thread in this phase \(through `CurrentUnitOfWork.set(UnitOfWork)`\). Subsequently, the message is typically handled by a message handler in this phase.
* Commit phase - after processing of the message is done but before the Unit of Work is committed, the `onPrepareCommit` listeners are invoked. If a Unit of Work is bound to a transaction, the `onCommit` listeners are invoked to commit any supporting transactions. When the commit succeeds, the `afterCommit` listeners are invoked. If a commit or any step before fails, the `onRollback` listeners are invoked. The message handler result is contained in the `ExecutionResult` of the unit of work, if available.
* Cleanup phase - the phase where any of the resources held by this unit of work \(such as locks\) are to be released. If multiple units of work are nested, the cleanup phase is postponed until the outer unit of work is ready to clean up.

The message handling process can be considered an atomic procedure; it should either be processed entirely, or not at all. Axon Framework uses the unit of work to track actions performed by the message handlers. After the handler completed, Axon will try to commit the actions registered with the unit Of work.

It is possible to bind a transaction to a unit of work. Many components, such as the `CommandBus` and `QueryBus` implementations and all asynchronously processing `EventProcessor`s, allow you to configure a `TransactionManager`. This transaction manager will then be used to create the transactions to bind to the unit of work that is used to manage the process of a message.

When application components need resources at different stages of message processing, such as a database connection or an `EntityManager`, these resources can be attached to the `UnitOfWork`. The `unitOfWork.getResources()` method allows you to access the resources attached to the current unit of work. Several helper methods are available on the unit of work directly, to make working with resources easier.

When nested units of work need to be able to access a resource, it is recommended to register it on the root unit of work, which can be accessed using `unitOfWork.root()`. If a unit of work is the root, it will simply return itself.

## Correlation Data Provider

As already mentioned, Axon Framework is based on messaging concepts. In order to understand the flow of a living system, it is crucial to follow the message path. Hence, specific meta-data should be added to each message to be able to track its path through the system. Correlation Data is something that is usually added to a message its meta-data so that we can track the origin and causality of the message.

Axon defines the `CorrelationDataProvider` as the mechanism to extract the Correlation Data from a given Message. There are three implementations of `CorrelationDataProvider` that come with Axon:

* `MessageOriginProvider` - Provides the correlation identifier and trace identifier based on configurable keys in meta-data. The Correlation Identifier identifies the current message, whilst the Trace Identifier is the identifier of the message which started the current processing \(note: this could be the identifier of the current message if the current message starts the processing\). Processing in this context can be understood as a business transaction. This is the default `CorrelationDataProvider`.
* `SimpleCorrelationDataProvider` - This is a broader provider than `MessageOriginProvider`, in the sense that it does not rely on Correlation and Trace Identifiers alone, but defines configurable headers \(similar to keys in `MessageOriginProvider`\) and based on them extracts information from the message's meta-data.
* `MultiCorrelationDataProvider` - A simple wrapper which has a list of `CorrelationDataProvider`s where it delegates the Correlation Data extraction to.

Besides using the mentoined set above, it is also possible to define your own `CorrelationDataProvider`.

On top of `CorrelationDataProvider`s there is a `CorrelationDataInterceptor` which is a handler interceptor and it registers all configured `CorrelationDataProvider`s with a current unit of work.

By calling `Configurer#configureCorrelationDataProviders(Function<Configuration, List<CorrelationDataProvider>> correlationDataProviderBuilder)` it is possible to configure `CorrelationDataProvider`s. If you are using Spring, having a bean of type `CorrelationDataProvider` in the Spring Application Context is sufficient.

