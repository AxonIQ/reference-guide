# Supported Parameters for Annotated Handlers

This chapter provides an exhaustive list of all the possible parameters for annotated message handling functions. The parameters for any message handling function are resolved through an Axon Framework internal mechanism, called the `ParameterResolver`.

The set of `ParameterResolver`s can be extended if custom \(or not yet\) supported parameters should be injected in to your annotated handlers. For more specifics on configuring custom `ParameterResolver`s we suggest reading [this](../../appendices/message-handler-tuning/parameter-resolvers.md) section.

## Supported Parameters for Command Handlers

By default, `@CommandHandler` annotated methods allow the following parameter types:

* The first parameter is the payload of the command message.

  It may also be of type `Message` or `CommandMessage`, if the `@CommandHandler` annotation explicitly defined the name of the command the handler can process.

  By default, a command name is the fully qualified class name of the command its payload.

* Parameters annotated with `@MetaDataValue` will resolve to the metadata value with the key as indicated on the annotation.

  If `required` is `false` \(default\), `null` is passed when the metadata value is not present.

  If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.

* Parameters of type `MetaData` will have the entire `MetaData` of a `CommandMessage` injected.
* Parameters of type `UnitOfWork` get the current unit of work injected. This allows command handlers to register actions to be performed at specific stages of the Unit of Work, or gain access to the resources registered with it.
* Parameters of type `Message`, or `CommandMessage` will get the complete message, with both the payload and the metadata.

  This is useful if a method needs several metadata fields, or other properties of the wrapping Message.

* A parameter of type `String` annotated with `@MessageIdentifier` will resolve the identifier of the `CommandMessage` being handled
* Parameters of type `ConflictResolver` will resolve the configured `ConflictResolver` instance.

  See the [Conflict Resolution](../axon-framework-commands/modeling/conflict-resolution.md) section for specifics on this topic.

* Parameters of type `InterceptorChain` will resolve the chain of `MessageHandlerInterceptor`s for a `CommandMessage`.

  This feature should be used in conjunction with a `@CommandHandlerInterceptor` annotated method.

  For more specifics on this it is recommend to read [this](message-intercepting.md#commandhandlerinterceptor-annotation) section.

* If the application is run in a Spring environment, any Spring Bean can be resolved.

  Note that the `@Qualifier` annotation can be used in conjunction with this to further specify which Bean should be resolved.

## Supported Parameters for Query Handlers

By default, `@QueryHandler` annotated methods allow the following parameter types:

* The first parameter is the payload of the query message.

  It may also be of type `Message` or `QueryMessage`, if the `@QueryHandler` annotation explicitly defined the name of the query the handler can process.

  By default, a query name is the fully qualified class name of the query its payload.

* Parameters annotated with `@MetaDataValue` will resolve to the metadata value with the key as indicated on the annotation.

  If `required` is `false` \(default\), `null` is passed when the metadata value is not present.

  If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.

* Parameters of type `MetaData` will have the entire metadata of a `QueryMessage` injected.
* Parameters of type `UnitOfWork` get the current unit of work injected. This allows query handlers to register actions to be performed at specific stages of the unit of work, or gain access to the resources registered with it.
* Parameters of type `Message`, or `QueryMessage` will get the complete message, with both the payload and the metadata.

  This is useful if a method needs several meta data fields, or other properties of the wrapping message.

* A parameter of type `String` annotated with `@MessageIdentifier` will resolve the identifier of the `QueryMessage` being handled
* If the application is run in a Spring environment, any Spring Bean can be resolved.

  Note that the `@Qualifier` annotation can be used in conjunction with this to further specify which Bean should be resolved.

