# Supported Parameters for Annotated Handlers

This chapter provides an exhaustive list of all the possible parameters for annotated message handling functions. 
The framework resolves the parameters for any message handling function through an internal mechanism, called the `ParameterResolver`.
The `ParameterResolver`, built by a `ParameterResolverFactory`, is in charge of inserting the parameters for the command, event and query handlers.

The set of `ParameterResolver`s can be extended if custom \(or not yet\) supported parameters should be injected in to your annotated handlers.
You can configure additional `ParameterResolver`s by implementing the `ParameterResolverFactory` interface and configuring the new implementation.
For more specifics on configuring custom `ParameterResolver`s we suggest reading [this](../../appendices/message-handler-tuning/parameter-resolvers.md) section.

## Supported Parameters for Command Handlers

By default, `@CommandHandler` annotated methods allow the following parameter types:

* The first parameter is always the payload of the command message.
  It may also be of type `Message` or `CommandMessage`, if the `@CommandHandler` annotation explicitly defined the name of the command the handler can process.
  By default, a command name is the fully qualified class name of the command its payload.

* Parameters of type `MetaData` will have the entire metadata of a `CommandMessage` injected.

* Parameters annotated with `@MetaDataValue` will resolve the metadata value with the key as indicated on the annotation.
  If `required` is `false` \(default\), `null` is passed when the metadata value is not present.
  If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.

* Parameters of type `Message`, or `CommandMessage` will get the complete message, with both the payload and the metadata.
  Resolving the entire `Message` is helpful if a method needs several metadata fields or other properties of the message.

* Parameters of type `UnitOfWork` get the current [unit of work](unit-of-work.md) injected.
  The `UnitOfWork` allows command handlers to register actions to be performed at specific stages of the unit of work or gain access to the resources registered with it.

* A parameter of type `String` annotated with `@MessageIdentifier` will resolve the identifier of the handled `CommandMessage`.

* Parameters of type `ConflictResolver` will resolve the configured `ConflictResolver` instance.
  See the [Conflict Resolution](../axon-framework-commands/modeling/conflict-resolution.md) section for specifics on this topic.

* Parameters of type `InterceptorChain` will resolve the chain of `MessageHandlerInterceptor`s for a `CommandMessage`.
  You should use this feature in conjunction with a `@CommandHandlerInterceptor` annotated method.
  For more specifics on this it is recommended to read [this](message-intercepting.md#commandhandlerinterceptor-annotation) section.

* The parameter resolvers can resolve a `ScopeDescriptor` too.
  The scope descriptor is helpful when [scheduling a deadline](../deadlines/README.md) through the `DeadlineManager`.
  Note that the `ScopeDescriptor` only makes sense from within the scope of an Aggregate or Saga.

* If the application runs in a Spring environment, any Spring Bean can be resolved.
  To that end, we should annotate the desired Spring bean with `@Autowired`.
  We can extend the annotation with `@Qualifier` if a specific version of the bean should be wired.

## Supported Parameters for Event Handlers

By default, `@EventHandler` annotated methods allow the following parameter types:

* The first parameter is the payload of the event message.
  If the event handler does not need access to the payload of the message, you can specify the expected payload type on the `@EventHandler` annotation.
  Do not configure the payload type on the annotation if you want the payload passed as a parameter.

* Parameters of type `MetaData` will have the entire metadata of an `EventMessage` injected.

* Parameters annotated with `@MetaDataValue` will resolve the metadata value with the key as indicated on the annotation.
  If `required` is `false` \(default\), `null` is passed when the metadata value is not present.
  If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.

* We can resolve the `EventMessage` in its entirety as well.
  If the first parameter is of type message, it effectively matches an event of any type, even if generic parameters suggest otherwise.
  Due to type erasure, Axon cannot detect what parameter the implementation expects.
  It is best to declare a parameter of the payload type in such a case, followed by a parameter of type message.

* Parameters of type `UnitOfWork` get the current [unit of work](unit-of-work.md) injected.
  The `UnitOfWork` allows event handlers to register actions to be performed at specific stages of the unit of work or gain access to the resources registered with it.

* A parameter of type `String` annotated with `@MessageIdentifier` will resolve the identifier of the handled `EventMessage`.

* Parameters annotated with `@Timestamp` and of type `java.time.Instant` \(or `java.time.temporal.Temporal`\) will resolve to the timestamp of the `EventMessage`.
  The resolved timestamp is the time at which the event was generated.

* Parameters annotated with `@SequenceNumber` and of type `java.lang.Long` or `long` will resolve to the `sequenceNumber` of a `DomainEventMessage`.
  This parameter provides the order in which the event was generated (within the aggregate scope it originated from).
  It is important to note that `DomainEventMessage` **can only** originate from an Aggregate.
  Hence, events that have been published directly on the `EventBus`/`EventGateway` are _not_ implementations of the `DomainEventMessage`. As such, they will not resolve a sequence number.

* Parameters of type `TrackingToken` will have the current [token](../events/event-processors/streaming.md#tracking-tokens) related to the processed event injected.
  Note that this will only work for `StreamingEventProcessor` instances, as otherwise, there is no token attached to the events.

* Parameters annotated with `@SourceId` and of type `java.lang.String` will resolve to the `aggregateIdentifier` of a `DomainEventMessage`.
  This parameter provides the identifier of the aggregate from which the event originates.
  It is important to note that `DomainEventMessage` **can only** originate from an Aggregate.
  Hence, events that have been published directly on the `EventBus`/`EventGateway` are _not_ implementations of the `DomainEventMessage`. As such, they will not resolve a source id.

* If the application runs in a Spring environment, any Spring Bean can be resolved.
  To that end, we should annotate the desired Spring bean with `@Autowired`.
  We can extend the annotation with `@Qualifier` if a specific version of the bean should be wired.

## Supported Parameters for Query Handlers

By default, `@QueryHandler` annotated methods allow the following parameter types:

* The first parameter is always the payload of the query message.
  It may also be of type `Message` or `QueryMessage`, if the `@QueryHandler` annotation explicitly defined the name of the query the handler can process.
  By default, a query name is the fully qualified class name of the query its payload.

* Parameters of type `MetaData` will have the entire metadata of a `QueryMessage` injected.

* Parameters annotated with `@MetaDataValue` will resolve the metadata value with the key as indicated on the annotation.
  If `required` is `false` \(default\), `null` is passed when the metadata value is not present.
  If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.

* Parameters of type `Message`, or `QueryMessage` will get the complete message, with both the payload and the metadata.
  Resolving the entire `Message` is helpful if a method needs several metadata fields or other properties of the message.

* Parameters of type `UnitOfWork` get the current [unit of work](unit-of-work.md) injected.
  The `UnitOfWork` allows query handlers to register actions to be performed at specific stages of the unit of work or gain access to the resources registered with it.

* A parameter of type `String` annotated with `@MessageIdentifier` will resolve the identifier of the handled `CommandMessage`.

* If the application runs in a Spring environment, any Spring Bean can be resolved.
  To that end, we should annotate the desired Spring bean with `@Autowired`.
  We can extend the annotation with `@Qualifier` if a specific version of the bean should be wired.
