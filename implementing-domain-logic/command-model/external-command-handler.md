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