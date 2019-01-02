# Dispatching commands

TODO
- CommandMessage
- CommandBus
- CommandGateway
- CommandCallback
- CommandResultMessage
e.g.: 
    When a command creates an aggregate instance, the callback for that command will receive the aggregate identifier when the command executed successfully.
and:
    ## Returning results from Command Handlers
    
    In some cases, the component dispatching a command needs information about the processing results of a command. 
    A command handler method can return a value from its method. 
    That value will be provided to the sender as the result of the command.
    
    One exception is the `@CommandHandler` on an aggregate's constructor. 
    In this case, instead of returning the return value of the method \(which is the Aggregate itself\), 
     the value of the `@AggregateIdentifier` annotated field is returned.
    
    > **Note**
    >
    > While it is possible to return results from commands, it should be used sparsely. 
    > The intent of the Command should never be to retrieve a value,
    >  as that would be an indication that the message should be designed as a [Query Message](../query-handling/query-handling.md). 
    > A typical example for a Command result is the identifier of a newly created entity.
