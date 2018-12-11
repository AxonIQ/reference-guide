# External command handler

In certain cases, it is not possible, or desired to route a command directly to an aggregate instance. In such case, it is possible to register a command handler object.

A command handler object is a simple \(regular\) object, which has `@CommandHandler` annotated methods. Unlike in the case of an aggregate, there is only a single instance of a command handler object, which handles all commands of the types it declares in its methods.

```java
public class MyAnnotatedHandler {

    @CommandHandler
    public void handleSomeCommand(SomeCommand command, @MetaDataValue("userId") String userId) {
        // whatever logic here
    }

    @CommandHandler(commandName = "myCustomCommand")
    public void handleCustomCommand(SomeCommand command) {
       // handling logic here
    }

}

// To register the annotated handlers to the command bus:
Configurer configurer = ...
configurer.registerCommandHandler(c -> new MyAnnotatedHandler());
```

### Returning results from command handlers

In some cases, the component dispatching a command needs information about the processing results of a command. A command handler method can return a value from its method. That value will be provided to the sender as the result of the command.

One exception is the `@CommandHandler` on an aggregate's constructor. In this case, instead of returning the return value of the method \(which is the Aggregate itself\), the value of the `@AggregateIdentifier` annotated field is returned instead

> **Note**
>
> While it is possible to return results from commands, it should be used sparsely. The intent of the command should never be in getting a value, as that would be an indication the message should be designed as a [Query Message ](https://docs.axoniq.io/reference-guide/1.2-domain-logic/query-handling)instead. A typical example for a Command result is the identifier of a newly created entity.