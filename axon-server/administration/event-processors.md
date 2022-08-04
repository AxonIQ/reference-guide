# Event Processors

## How to reset the Token of an Event Processor

It might be desirable to reset the position an event processor works on.
This causes a replay of the events for this event processor.
Resetting the position is done by resetting the token of the event processor.

We document the following ways to reset the token of an event processor:
- Using Axon Framework
- Using the REST API
- Using the Axon Server Connector

A minimal project showing these approaches can be found [here](https://github.com/AxonIQ/code-samples/tree/master/reset-handler).

### General
Independent of the method chosen, an event processor needs to be stopped before its token can be reset.
This requires all instances of the event processor (on all nodes) to release their claims on segments.
Now, when `resetTokens()` is called, the executing instance can temporarily claim all segments, reset the tokens and store the new values.
After the token has been reset, the event processor can be started again.

Note that older versions of axon framework (pre 4.6) directly acknowledge a received admin instruction, without waiting for its execution.
This might require manually waiting for an Event Processor to be fully stopped before the token can be reset.
A possible approach for this is shown in the last section of this page.

### Using Axon Framework
Axon Framework exposes all required functionality for a reset in the `StreamingEventProcessor` class. In the following, we provide a sample for its usage. For more details, read the documentation on the framework classes [here](../../axon-framework/events/event-processors/streaming.md#triggering-a-reset).
Instances of `StreamingEventProcessor` can be obtained by querying the `eventProcessingConfiguration` method of your global configuration.
```java
import org.axonframework.config.Configuration;
// …
private final Configuration configuration;
// …

configuration.eventProcessingConfiguration()
    .eventProcessorByProcessingGroup(processorName,
        StreamingEventProcessor.class)
    // …
```
Calling `shutDown`, `resetTokens` and `start` on the retrieved `StreamingEventProcessor` performs the required steps to reset its token.
This can be achieved as shown in the following code snippet:
```java
configuration.eventProcessingConfiguration()
    .eventProcessorByProcessingGroup(processorName,
        StreamingEventProcessor.class)
    .ifPresent(streamingEventProcessor -> {
        if (streamingEventProcessor.supportsReset()) {
            streamingEventProcessor.shutDown();
            streamingEventProcessor.resetTokens();
            streamingEventProcessor.start();
        }
    });
```

Note that this only concerns stopping the local instances. 
If there are instances running on other nodes, you either need to use Axon Server or build a solution for this yourself.
For more details, refer to the section in the Axon Framework [reference guide](../../axon-framework/events/event-processors/streaming.md#triggering-a-reset).

### Using the REST API
Axon Server exposes a REST API to pause and start Event Processors.
When used in conjunction with the `resetTokens` method from earlier, this can be used to make sure that all instances of an Event Processor are paused before a token is reset.
The required parameters to do this are the following:
- *component*: name of the component the Event Processor is part of
- *processor*: name of the Event Processor itself
- *context*: name of the context for which to reset the tokens
- *tokenStoreId*: the identifier used to distinguish the desired token from other tokens stored in the same store. 

Now 
`/v1/components/{component}/processors/{processor}/pause?context={context}&tokenStoreIdentifier={tokenStoreId}`
can be called, causing Axon Server to request all matched Event Processors to stop.
Then a reset as shown in [the example using the framework](event-processors.md#using-axon-framework)
 can be performed.
This is not Axon Server specific and hence is the same for all shown methods.

As a final step, the Event Processors can be started again with a patch request to the following url:
`/v1/components/{component}/processors/{processor}/start?context={context}&tokenStoreIdentifier={tokenStoreId}` .


### Using the Axon Server Connector
To reset Event Processors with the Axon Server Connector, the dependency has to be available on your classpath, 
e.g. by using the following maven dependency.
```xml
<dependency>
    <groupId>io.axoniq</groupId>
    <artifactId>axonserver-connector-java</artifactId>
    <version>4.6.1</version>
</dependency>
```

All operations related to administration go through an `AdminChannel`, 
which can be obtained from an `AxonServerConnectionFactory` as follows.

```java
private AdminChannel adminChannel() {
    AxonServerConnectionFactory connectionFactory = AxonServerConnectionFactory.forClient(componentName).build();
    return connectionFactory.connect(contextName).adminChannel();
}
```

In this example, `componentName` and `contextName` are values supplied by external configuration.
In simple cases, these might be the same as in your Axon Framework configuration.

Using the provided admin channel, you can pause, reset and restart the event processors.
In contrast to the approach based on the Axon Framework, you also need to provide a `tokenStoreIdentifier`, 
since there can be multiple applications connected to one Axon Server that share the same token store. 
An example on how to get this identifier can be found in the Axon Framework Documentation on [Retrieving the Token Store Identifier](../../axon-framework/events/event-processors/streaming.md#retrieving-the-token-store-identifier).

Now resetting the tokens can be done by simply calling the `pauseEventProcessor`, `resetTokens` and `startEventProcessor` 
methods in the correct order.

```java
adminChannel().pauseEventProcessor(eventProcessorName, tokenStoreIdentifier)
        .thenRun(eventProcessor::resetTokens)
        .thenRun(() -> adminChannel().startEventProcessor(eventProcessorName, tokenStoreIdentifier))
```

Note that Axon Server makes sure to stop and start all matched EventProcessors on all nodes with this call. 


### Handling asynchronous behaviour in older Axon Framework versions (pre 4.6)

Prior to version 4.6, Axon Framework did immediately acknowledge receiving a pause instruction.
This means, that the Axon Server would receive these `ACCEPTED` Results before all Event Processors have terminated.

Starting from Axon Framework version 4.6, Axon Server will only respond with `SUCCESS` once all connected Event Processors have successfully been paused.
To make sure that older Axon Framework versions wait until all Event Processors have terminated, we need to implement a bit of custom logic.
This works with both, the Axon Server Connector approach and the REST API approach. 
In the following, we will show an approach using the axon server connector admin channel.

We use the following method to allow waiting for all instances of an event processor, identified by a tuple of 
`eventProcessorName` and `tokenStoreIdentifier` to reach a desired state, either running or not running.
Since APIs are built around an asynchronous execution model, we work with `Mono` and `Flux` here.

```java
    protected Mono<Result> awaitForStatus(String eventProcessorName, String tokenStoreIdentifier, boolean running) {
        return Flux.from(new ResultStreamPublisher<>(adminChannel::eventProcessors))
                   .filter(eventProcessor -> eventProcessor.getIdentifier().getProcessorName()
                                                           .equals(eventProcessorName))
                   .filter(eventProcessor -> eventProcessor.getIdentifier().getTokenStoreIdentifier()
                                                           .equals(tokenStoreIdentifier))
                   .flatMap(eventProcessor -> Flux.fromIterable(eventProcessor.getClientInstanceList()))
                   .map(clientInstance -> clientInstance.getIsRunning() == running)
                   .reduce(Boolean::logicalAnd)
                   .filter(result -> result.equals(true))
                   .switchIfEmpty(Mono.error(new RuntimeException("")))
                   .retryWhen(Retry.fixedDelay(3, Duration.ofSeconds(2)))
                   .thenReturn(Result.SUCCESS);
    }
```
The basic idea is to filter all received event processor descriptions to only retain the relevant ones, 
get a list of all connected client nodes and ensure their state equals the desired state passed in as a parameter.
If one of them is in the wrong state, the check is repeated at most 3 times with a fixed delay of 2 seconds.
If there are still clients in the wrong state, an error is returned.

Using an approach like this allows you to build custom functionality to handle unresponsive clients in a tailor-made solution.