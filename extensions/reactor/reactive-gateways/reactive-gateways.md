# Reactive Gateways

The "Reactive Gateways" offer a reactive API wrapper around the command, query and event bus.
Most of the operations are similar to those from non-reactive gateways, simply replacing the `CompletableFuture` with either a `Mono` or `Flux`.
In some cases, the API is expended to ease use of common reactive patterns.

{% hint style="info" %}
Reactor doesn't allow `null` values in streams.
Any null value returned from the handler will be mapped to [Mono#empty()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#empty).
{% endhint %}

> **Retrying operations**
> 
> All operations support Reactor's retry mechanism:
> 
> `reactiveQueryGateway.query(query, ResponseType.class).retry(5);`
>
> This call will retry sending the query a maximum of five times when it fails.

## Configuration in Spring Boot

This extension can be added as a Spring Boot starter dependency to your project using group id `org.axonframework.extensions.reactor` and artifact id `axon-reactor-spring-boot-starter`.
The implementation of the extension can be found [here](https://github.com/AxonFramework/extension-reactor).


## Reactor Command Gateway

This section describes the methods on the `ReactorCommandGateway`.

**`send`** - Sends the given command once the caller subscribes to the command result. Returns immediately.

A common pattern is using the REST API to send a command. 
In this case it is recommend to for example use [WebFlux](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html), and return the command result `Mono` directly to the controller:

```java
class SpringCommandController {

    private final ReactorCommandGateway reactiveCommandGateway; 
    
    @PostMapping
    public Mono<CommandHandlerResponseBody> sendCommand(@RequestBody CommandBody command) {
        return reactiveCommandGateway.send(command);
    }
}
```
_Sending a command from a Spring WebFlux Controller._

{% hint style="info" %}
If the command handling function returns type `void`, `Mono<CommandHandlerResponseBody>` should be replaced with `Mono<Void>`
{% endhint %}

Another common pattern is "send and forget":  

```java
class CommandDispatcher {

    private final ReactorCommandGateway reactiveCommandGateway;
    
    public void sendAndForget(MyCommand command) {
         reactiveCommandGateway.send(command)
                               .subscribe();
    }
}
```
_Function that sends a command and returns immediately without waiting for the result._

**`sendAll`** - This method uses the given `Publisher` of commands to dispatch incoming commands.

{% hint style="info" %}
This operation is available only in the Reactor extension. Use it to connect 3rd party streams that delivers commands.
{% endhint %}

```java
class CommandPublisher {

    private final ReactorCommandGateway reactiveCommandGateway;
    
    @PostConstruct
    public void startReceivingCommands(Flux<CommandBody> inputStream) {
        reactiveCommandGateway.sendAll(inputStream)
                              .subscribe();
    }
}
```
_Connects an external input stream directly to the Reactor Command Gateway._

{% hint style="info" %}
The `sendAll` operation will keep sending commands until the input stream is canceled. 
{% endhint %}

{% hint style="warn" %}
`send` and `sendAll` do not offer _any_ backpressure, yet. 
The only backpressure mechanism in place is that commands will be sent sequentially; thus once the result of a previous command arrives.
The number of commands is prefetched from an incoming stream and stored in a buffer for sending (see [Flux#concatMap](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concatMap)).
**This slows down sending**, but does not guarantee that the Subscriber will not be overwhelmed with commands if they are sent too fast.
{% endhint %}

## Reactor Query Gateway

**`query`** - Sends the given `query`, expecting a response in the form of `responseType` from a single source.

```java
class SpringQueryController {
    
    private final ReactorQueryGateway reactiveQueryGateway;

    // The query's Mono is returned to the Spring controller. Subscribe control is given to Spring Framework.
    @GetMapping
    public Mono<SomeResponseType> findAll(FindAllQuery query, Class<SomeResponseType> responseType) {
        return reactiveQueryGateway.query(query, responseType);
    }
}
```
_Recommended way of using the Reactor query gateway within a Spring REST controller._

**`scatterGather`** - Sends the given `query`, expecting a response in the form of `responseType` from several sources within a specified `duration`. 

```java
class SpringQueryController {
    
    private final ReactorQueryGateway reactiveQueryGateway;

    @GetMapping
    public Flux<SomeResponseType> findMany(FindManyQuery query) {
        return reactiveQueryGateway.scatterGather(query, SomeResponseType.class, Duration.ofSeconds(5)).take(3);
    }
}
```
_Sends a given query that stops after receiving three results, or after 5 seconds._

### Subscription queries

Firstly, the Reactor API for subscription queries in Axon is not new. 

However, we noticed several patterns which are often used, such as:
 
 * Concatenating initial results with query updates in a single stream, or
 * skipping the initial result all together.
 
As such, the Reactor Extension provides several methods to ease usage of these common patterns. 

**`subscriptionQuery`** - Sends the given `query`, returns the initial result and keeps streaming incremental updates until a subscriber unsubscribes from the `Flux`.

Note that this method should be used when the response type of the initial result and incremental update match.

```java 
Flux<ResultType> resultFlux = reactiveQueryGateway.subscriptionQuery("criteriaQuery", ResultType.class);
```

The above invocation through the `ReactorQueryGateway` is equivalent to: 

```java
class SubscriptionQuerySender {
    
    private final ReactorQueryGateway reactiveQueryGateway;
    
    public Flux<SomeResponseType> sendSubscriptionQuery(SomeQuery query, Class<SomeResponseType> responseType) {
        return reactiveQueryGateway.subscriptionQuery(query, responseType, responseType)
                                   .flatMapMany(result -> result.initialResult()
                                                                .concatWith(result.updates())
                                                                .doFinally(signal -> result.close()));
    }   
}
```

**`subscriptionQueryMany`** - Sends the given `query`, returns the initial result and keeps streaming incremental updates until a subscriber unsubscribes from the `Flux`.

This operation should be used when the initial result contains multiple instances of the response type which needs to be flattened.
Additionally, the response type of the initial response and incremental updates need to match.

```java   
Flux<ResultType> resultFlux = reactiveQueryGateway.subscriptionQueryMany("criteriaQuery", ResultType.class);
```

The above invocation through the `ReactorQueryGateway` is equivalent to: 

```java
class SubscriptionQuerySender {
    
    private final ReactorQueryGateway reactiveQueryGateway;
    
    public Flux<SomeResponseType> sendSubscriptionQuery(SomeQuery query, Class<SomeResponseType> responseType) {
        return reactiveQueryGateway.subscriptionQuery(query,
                                                      ResponseTypes.multipleInstancesOf(responseType),
                                                      ResponseTypes.instanceOf(responseType))
                                   .flatMapMany(result -> result.initialResult()
                                                                .flatMapMany(Flux::fromIterable)
                                                                .concatWith(result.updates())
                                                                .doFinally(signal -> result.close()));
    }
}
```

**`queryUpdates`** - Sends the given `query` and streams incremental updates until a subscriber unsubscribes from the `Flux`.

This method could be used when subscriber is only interested in updates.

```java   
Flux<ResultType> updatesOnly = reactiveQueryGateway.queryUpdates("criteriaQuery", ResultType.class);
```

The above invocation through the `ReactorQueryGateway` is equivalent to:  

```java
class SubscriptionQuerySender {
    
    private final ReactorQueryGateway reactiveQueryGateway;
    
    public Flux<SomeResponseType> sendSubscriptionQuery(SomeQuery query, Class<SomeResponseType> responseType) {
        return reactiveQueryGateway.subscriptionQuery(query, ResponseTypes.instanceOf(Void.class), responseType)
                                   .flatMapMany(result -> result.updates()
                                                                .doFinally(signal -> result.close()));
    }
}
```

{% hint style="info" %}
In the above shown methods, the subscription query is closed automatically after a subscriber has unsubscribed from the `Flux`. 
When using the regular `QueryGateway`, the subscription query needs to be closed manually however.
{% endhint %}

## Reactor Event Gateway

Reactive variation of the `EventGateway`. Provides support for reactive return types such as `Flux`.

**`publish`** - Publishes the given `events` once the caller subscribes to the resulting `Flux`.

This method returns events that were published. 
Note that the returned events may be different from those the user has published, granted an [interceptor](#interceptors) has been registered which modifies events.

```java
class EventPublisher {
    
    private final ReactorEventGateway reactiveEventGateway;
    
    // Register a dispatch interceptor to modify the event messages
    public EventPublisher() {
        reactiveEventGateway.registerDispatchInterceptor(
            eventMono -> eventMono.map(event -> GenericEventMessage.asEventMessage("intercepted" + event.getPayload()))
        );
    }
    
    public void publishEvent() {
        Flux<Object> result = reactiveEventGateway.publish("event");
    }   
}
```
_Example of dispatcher modified events, returned to user as the result `Flux`._

## Interceptors

Axon provides a notion of [interceptors](../../../axon-framework/messaging-concepts/message-intercepting.md).
The Reactor gateways allow for similar interceptor logic, namely the `ReactorMessageDispatchInterceptor` and `ReactorResultHandlerInterceptor`.

These interceptors allow us to centrally define rules and filters that will be applied to a message stream.

{% hint style="info" %}
Interceptors will be applied in order they have been registered to the given component.
{% endhint %}

### Reactor Dispatch Interceptors

The `ReactorMessageDispatchInterceptor` should be used to centrally apply rules and validations for outgoing messages.
Note that a `ReactorMessageDispatchInterceptor` is an implementation of the default `MessageDispatchInterceptor` interface used throughout the framework.
The implementation of this interface is described as follows:

```java
@FunctionalInterface
public interface ReactorMessageDispatchInterceptor<M extends Message<?>> extends MessageDispatchInterceptor<M> {

    Mono<M> intercept(Mono<M> message);

    @Override
    default BiFunction<Integer, M, M> handle(List<? extends M> messages) {
        return (position, message) -> intercept(Mono.just(message)).block();
    }
}
```

It thus defaults the `MessageDispatchInterceptor#handle(List<? extends M>` method to utilize the `ReactorMessageDispatchInterceptor#intercept(Mono<M>)` method.
As such, a `ReactorMessageDispatchInterceptor` could thus be configured on a plain Axon gateway too.
Here are a couple of examples how a message dispatch interceptor could be used:

```java
class ReactorConfiguration {

    public void registerDispatchInterceptor(ReactorCommandGateway reactiveGateway) {
        reactiveGateway.registerDispatchInterceptor(
            msgMono -> msgMono.map(msg -> msg.andMetaData(Collections.singletonMap("key1", "value1")))
        );
    }
}
```
_Dispatch interceptor that adds key-value pairs to the message's `MetaData`._

```java
class ReactorConfiguration {

    public void registerDispatchInterceptor(ReactorEventGateway reactiveGateway) {
        reactiveGateway.registerDispatchInterceptor(
            msgMono -> msgMono.filterWhen(v -> Mono.subscriberContext()
                              .filter(ctx-> ctx.hasKey("security"))
                              .map(ctx->ctx.get("security")))
        );
    }
}
```
_Dispatch interceptor that discards the message, based on a security flag in the Reactor Context._

### Reactor Result Handler Interceptors

The `ReactorResultHandlerInterceptor` should be used  to centrally apply rules and validations for incoming messages, a.k.a. results.
The implementation of this interface is described as follows:

```java
@FunctionalInterface
public interface ReactorResultHandlerInterceptor<M extends Message<?>, R extends ResultMessage<?>> {
    
    Flux<R> intercept(M message, Flux<R> results);
}
```

The parameters are the `message` that has been sent, and a `Flux` of `results` from that message, which is going to be intercepted.
The `message` parameter can be useful if you want to apply a given result rule only for specific messages.
Here are a couple of examples how a message result interceptor could be used:

{% hint style="info" %}
This type of interceptor is available _only_ in the Reactor Extension.
{% endhint %}

```java
class ReactorConfiguration {

    public void registerResultInterceptor(ReactorQueryGateway reactiveGateway) {
        reactiveGateway.registerResultHandlerInterceptor(
            (msg, results) -> results.filter(r -> !r.getPayload().equals("blockedPayload"))
        );
    }
}
```
_Result interceptor which discards all results that have a payload matching `blockedPayload`_

```java
class ReactorConfiguration {

    public void registerResultInterceptor(ReactorQueryGateway reactiveGateway) {
        reactiveQueryGateway.registerResultHandlerInterceptor(
            (query, results) -> results.flatMap(r -> {
                if (r.getPayload().equals("")) {
                    return Flux.<ResultMessage<?>>error(new RuntimeException("no empty strings allowed"));
                } else {
                    return Flux.just(r);
                }
            })
        );
    }
}
```
_Result interceptor which validates that the query result does not contain an empty `String`._ 

```java
class ReactorConfiguration {

    public void registerResultInterceptor(ReactorQueryGateway reactiveGateway) {
        reactiveQueryGateway.registerResultHandlerInterceptor(
            (q, results) -> results.filter(it -> !((boolean) q.getQueryName().equals("myBlockedQuery")))
        );
    }
}
```
_Result interceptor which discards all results where the `queryName` matches `myBlockedQuery`._

```java
class ReactorConfiguration {

    public void registerResultInterceptor(ReactorCommandGateway reactiveGateway) {
        reactiveGateway.registerResultHandlerInterceptor(
            (msg,results) -> results.timeout(Duration.ofSeconds(30))
        );
    }
}
```
_Result interceptor which limits the result waiting time to thirty seconds per message._ 

```java
class ReactorConfiguration {

    public void registerResultInterceptor(ReactorCommandGateway reactiveGateway) {
        reactiveGateway.registerResultHandlerInterceptor(
            (msg,results) -> results.log().take(5)
        );
    }
}
```
_Result interceptor which limits the number of results to five entries, and logs all results._
