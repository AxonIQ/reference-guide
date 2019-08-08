# Supported Parameters for Annotated Handlers

By default, `@CommandHandler` annotated methods allow the following parameter types:

* The first parameter is the payload of the command message. It may also be of type `Message` or `CommandMessage`, if the `@CommandHandler` annotation explicitly defined the name of the command the handler can process. 
By default, a command name is the fully qualified class name of the command its payload.
* Parameters annotated with `@MetaDataValue` will resolve to the metadata value with the key as indicated on the annotation. If `required` is `false` \(default\), `null` is passed when the metadata value is not present. 
If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.
* Parameters of type `MetaData` will have the entire `MetaData` of a `CommandMessage` injected.
* Parameters of type `UnitOfWork` get the current unit of work injected. This allows command handlers to register actions to be performed at specific stages of the Unit of Work, or gain access to the resources registered with it.
* Parameters of type `Message`, or `CommandMessage` will get the complete message, with both the payload and the metadata. 
This is useful if a method needs several metadata fields, or other properties of the wrapping Message.
