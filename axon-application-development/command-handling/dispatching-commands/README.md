# Command Dispatchers

The [Aggregate](../modeling/aggregate.md) and [External Command Handler]() pages provide the background on how to handle command messages in your application. The dispatching process is the starting point of such a command message. Axon provides two interface you can use to send the commands to your command handlers, being:

1. The [Command Bus](./#the-command-bus), and
2. the [Command Gateway](./#the-command-gateway)

This page will show how and when to use the command gateway and bus. How to configure and specifics on the the command gateway and bus implementations are discussed [here](command-dispatching.md)

## The Command Bus

The 'Command Bus' is the mechanism that dispatches commands to their respective command handlers. As such it is the infrastructure component that is aware which component can handle which command.

Each command is always sent to exactly one command handler. If no command handler is available for the dispatched command, a `NoHandlerForCommandException` exception is thrown.

The `CommandBus` provides two methods to dispatch commands to their respective handler, being the `dispatch(CommandMessage)` and `dispatch(CommandMessage, CommandCallback)` methods:

```java
private CommandBus commandBus; // 1.

public void dispatchCommands() {
    String cardId = UUID.randomUUID().toString(); // 2.

    // 3. & 4.
    commandBus.dispatch(GenericCommandMessage.asCommandMessage(new IssueCardCommand(cardId, 100, "shopId")));

    // 5. & 6.
    commandBus.dispatch(
            GenericCommandMessage.asCommandMessage(new IssueCardCommand(cardId, 100, "shopId")),
            (CommandCallback<IssueCardCommand, String>) (cmdMsg, cmdResultMsg) -> {
                // 7.
                if (cmdResultMsg.isExceptional()) {
                    Throwable throwable = cmdResultMsg.exceptionResult();
                } else {
                    String commandResult = cmdResultMsg.getPayload();
                }
            }
    );
}
// omitted class, constructor and result usage
```

The `CommandDispatcher` described above exemplifies a couple of important aspects and capabilities of the dispatching commands:

1. The `CommandBus` interface providing the functionality to dispatch command messages.
2. The aggregate identifier is, per best practice, initialized as the String of a random unique identifier.

   Typed identifier objects are also possible, as long as the object implements a sensible `toString()` function.

3. The `GenericCommandMessage#asCommandMessage(Object)` method is used to create a `CommandMessage`. 

   To be able to dispatch a command on the `CommandBus`, you are required to wrap your own command object \(e.g. the 'command message payload'\) in a `CommandMessage`.

   The `CommandMessage` also allows the addition of [MetaData](../../messaging-concepts/message-anatomy.md#meta-data) to the Command Message.

4. The `CommandBus#dispatch(CommandMessage)` function will dispatch the provided `CommandMessage` on the bus, for delivery to a command handler. 

   If an application isn't directly interested in the outcome of a command, this method can be used.

5. If the outcome of command handling is relevant for your application, the optional second parameter can be provided, the `CommandCallback`.

   The `CommandCallback` allows the dispatching component to be notified when command handling is completed.

6. The Command Callback has one function, `onResult(CommandMessage, CommandResultMessage)`, which is called when command handling has finished. 

   The first parameter is the dispatched command, whilst the second is execution result of the dispatched command.

   Lastly, the `CommandCallback` is a 'functional interface' due to `onResult` being its only method.

   As such, `commandBus.dispatch(commandMessage, (cmdMsg, commandResultMessage) -> { /* ... */ })` would also be possible.

7. The `CommandResultMessage` provides the API to verify whether command execution was exceptional or successful. 

   If `CommandResultMessage#isExceptional` returns true, you can assume that the `CommandResultMessage#exceptionResult` will return a `Throwable` instance containing the actual exception.

   Otherwise, the `CommandResultMessage#getPayload` method _may_ provide you with an actual result or `null`, as further specified [here](./#command-dispatching-results).     

> **Command Callback consideration**
>
> In the case that `dispatch(CommandMessage, CommandCallback)` is used, the calling component may _not_ assume that the callback is invoked in the same thread that dispatched the command. If the calling thread depends on the result before continuing, you can use the `FutureCallback`. The `FutureCallback` is a combination of a `Future` \(as defined in the java.concurrent package\) and Axon's `CommandCallback`. Alternatively, consider using a `CommandGateway`.

## The Command Gateway

The 'Command Gateway' is a convenience approach towards dispatching commands. It does so by abstracting certain aspects for you when dispatching a command on the `CommandBus`. It this uses the `CommandBus` underneath to perform the actual dispatching of the message.  
While you are not required to use a gateway to dispatch commands, it is generally the easiest option to do so.

The `CommandGateway` interface can be separated in two sets of methods, namely `send` and `sendAndWait`:

```java
private CommandGateway commandGateway; // 1.

public void sendCommand() {
    String cardId = UUID.randomUUID().toString(); // 2.

    // 3.
    CompletableFuture<String> futureResult = commandGateway.send(new IssueCardCommand(cardId, 100, "shopId"));
}
// omitted class, constructor and result usage
```

The `send` API as shown above introduces a couple of concepts, marked with numbered comments:

1. The `CommandGateway` interface providing the functionality to dispatch command messages. 

   It does so by internally leveraging the `CommandBus` interface [dispatch messages](./#the-command-bus).

2. The aggregate identifier is, per best practice, initialized as the String of a random unique identifier.

   Typed identifier objects are also possible, as long as the object implements a sensible `toString()` function.

3. The `send(Object)` function requires a single parameter, the command object.

   This is an asynchronous approach to dispatching commands.

   As such the response of the `send` method is a `CompletableFuture`.

   This allows for chaining of follow up operations _after_ the command [result](./#command-dispatching-results) has been returned.

> **Callback when using `send(Object)`**
>
> The `CommandGateway#send(Object)` method uses the `FutureCallback` under the hood to unblock the command dispatching thread from the command handling thread.

A synchronous approach to sending messages can also be achieved, by using the `sendAndWait` methods:

```java
private CommandGateway commandGateway;

public void sendCommandAndWaitOnResult() {
    IssueCardCommand commandPayload = new IssueCardCommand(UUID.randomUUID().toString(), 100, "shopId");
    // 1.
    String result = commandGateway.sendAndWait(commandPayload);

    // 2.
    result = commandGateway.sendAndWait(commandPayload, 1000, TimeUnit.MILLISECONDS);
}
// omitted class, constructor and result usage
```

1. The `CommandGateway#sendAndWait(Object)` function takes in a single parameter, your command object.

   It will wait indefinitely until the command dispatching and handling process has been resolved.

   The result returned by this method can either be successful or exceptional, as will be explained [here](./#command-dispatching-results).

2. If waiting indefinitely is not desirable, a 'timeout' paired with the 'time unit' can be provided along side the command object.

   Doing so will ensure that the command dispatching thread will not wait longer than specified. 

   If command dispatching/handling was interrupted or the timeout was reached whilst using this approach, the command result will be `null`. 

   In all other scenarios, the result follows the [referenced](./#command-dispatching-results) approach.

## Command Dispatching Results

Dispatching commands will, generally speaking, have two possible outcomes:

1. Command handled successfully, and
2. command handled exceptionally

The outcome to some extent depends on the dispatching process, but more so on the implementation of the command handler. Thus if the `@CommandHandler` annotated [function](../modeling/aggregate.md#handling-commands-in-an-aggregate) throws an exception due to some business logic, it will be that exception which will be the result of dispatching the command.

The successful resolution of command handling intentionally _should not_ provide any return objects. Thus, if the `CommandBus`/`CommandGateway` provides a response \(either directly or through the `CommandResultMessage)`, then you should assume the result of successful command handling to return `null`.

While it is possible to return results from command handlers, this should be used sparsely. The intent of the Command should never be to retrieve a value, as that would be an indication that the message should be designed as a [Query Message](../../query-handling/). Exceptions to this would be the identifier of the Aggregate Root, or identifiers of entities the Aggregate Root has instantiated. The framework has one such exception build in, on the `@CommandHandler` annotated constructor of an Aggregate. In case the 'command handling constructor' has executed successfully, instead of the Aggregate itself, the value of the `@AggregateIdentifier` annotated field will be returned.

[Axon Coding Tutorial \#5: - Connecting the UI](https://youtu.be/lxonQnu1txQ)
