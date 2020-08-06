# Reactive Gateways

Reactive Gateways offers Reactive API wrapper around Command, Query and Event bus.
Most of the operations are similar to those from non-reactive gateways, where `CompletableFuture` is replaced with either Mono or Flux.
In some cases API is expended, to ease use of common patterns.


{% hint style="info" %}
Reactor doesn't allow `null` values in streams, any null value returned from the handler will be mapped to [Mono#empty()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#empty)
{% endhint %}


*Retries

All operation support Reactor's retry mechanism.

`reactiveQueryGateway.query(query, ResponseType.class).retry(5);`

_Retries sending of a query maximum 5 times if query fails._



## Command Gateway

**`send`** - Sends the given command once the caller subscribes to the command result. Returns immediately.

A common pattern is using REST API to send a command. In this case its recommend to use (WebFlux)[https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html],
and return command result Mono directly to the controller.

```java
     @PostMapping
     public Mono<CommandHandlerResponseBody> sendCommand(@RequestBody CommandBody command) {
         return reactiveCommandGateway
                 .send(command);
     }
```
_Sending a command from Spring Controller._

{% hint style="info" %}
If command handler is type of `void`, `Mono<CommandHandlerResponseBody>` should be replaced with `Mono<Void>`
{% endhint %}


Another common pattern is `send and forget`.  

```java
     public void sendAndForget(CommandBody command) {
          reactiveCommandGateway
                 .send(command)
                 .subscribe();
     }
```
_Functions that sends a command and returns immediately without waiting for result._


**`sendAll`** - Uses given Publisher of commands to send incoming commands away.

{% hint style="info" %}
This operation is available only in Reactor extension. Use it to connect 3th party streams that delivers commands.
{% endhint %}

```java
    Flux<CommandBody> inputStream = ...;

    @PostConstruct
    public void startReceivingCommands(){
        reactiveCommandGateway.sendAll(inputStream)
                .subscribe();
    }
```
_Connects external input stream directly to Command Gateway._

{% hint style="info" %}
`sendAll` will keep sending commands until input stream gets canceled. 
{% endhint %}

{% hint style="warn" %}
`send` & `sendAll` do not offer any backpressure yet. Only "backpressure" mechanism in place is that commands will be sent sequentially - once a result of previous command arrives.
Number of commands if prefetched from an incoming stream and stored in buffer for sending. See [Flux#concatMap](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concatMap)
**It slows down sending**, but does not guarantee that Subscriber will not be overwhelmed with commands if they are sent too fast.
{% endhint %}


## Query Gateway

**`query`** Sends the given query over expecting a response in the form of `responseType` from a single source.

```java
     @GetMapping
     public Mono<ResponseType> findAll() {
         return reactiveQueryGateway
                 .query(findAllQuery,ResponseType.class);
     }
```
_Recommended way of using query gateway within Spring REST controllers.
Query Mono is returned to Spring controller. Subscribe control is given to Spring Framework._

**`scatterGather`** Sends the given query over expecting a response in the form of `responseType` from several sources. 

```java
     @GetMapping
     public Flux<ResponseType> findMany() {
         return reactiveQueryGateway
                 .scatterGather(queries, 5, TimeUnit.SECONDS)
                 .take(3);
     }
```
_Sends a given query that stops after receiving 3 results or after 5 seconds._


### Subscription queries

Reactor API for subscription queries is not new. 

However, we noticed a several pattern often used, such as concatenating initial results with query updates in a single stream, or skip initial result all together, so we added several methods to ease usage of these common patterns. 

**`subscriptionQuery`** Sends the given query, returns initial result and keeps streaming incremental updates until a subscriber unsubscribes from Flux.

Should be used when response type of initial result and incremental update match.

```java 
Flux<String> resultFlux = reactiveQueryGateway.subscriptionQuery("criteriaQuery", ResultType.class);
```

is equivalent to 

```java
subscriptionQuery(query, resultType, resultType)
                   .flatMapMany(result -> result.initialResult()
                                                .concatWith(result.updates())
                                                .doFinally(signal -> result.close()));
```

**`subscriptionQueryMany`** Sends the given query, returns initial result and keeps streaming incremental updates until a subscriber unsubscribes from Flux.

Should be used when initial result contains multiple instances of response type and needs to be flatten.
Response type of initial response and incremental updates needs to match.

```java   
Flux<String> resultFlux = reactiveQueryGateway.subscriptionQueryMany("criteriaQuery", ResultType.class);
```

is equivalent to 

```java
subscriptionQuery(query,
                                 ResponseTypes.multipleInstancesOf(resultType),
                                 ResponseTypes.instanceOf(resultType))
                .flatMapMany(result -> result.initialResult()
                                             .flatMapMany(Flux::fromIterable)
                                             .concatWith(result.updates())
                                             .doFinally(signal -> result.close()));
```

**`queryUpdates`** sends the given query and streams incremental updates until a subscriber unsubscribes from Flux.

Should be used when subscriber has interest only in updates.

```java   
Flux<String> updatesOnly = reactiveQueryGateway.queryUpdates("criteriaQuery", ResultType.class);
```

is equivalent to 

```java
subscriptionQuery(query, ResponseTypes.instanceOf(Void.class), resultType)
                .flatMapMany(result -> result.updates()
                                             .doFinally(signal -> result.close()))
```

{% hint style="info" %}
In these methods' subscription query is closed automatically after a subscriber has unsubscribed from the Flux, where usually closing subscription query needs to be done manually.
{% endhint %}


## Event Gateway

Variation of EventGateway. Provides support for reactive return types such as Flux.

**`publish`** Publishes given events once the caller subscribes to the resulting Flux.

Returns events that were published. This means that returning events can be different from those users is publishing 
if user has registered interceptor that modifies events.


```java
        gateway.registerDispatchInterceptor(eventMono -> eventMono
                .map(event -> GenericEventMessage.asEventMessage("intercepted" + event.getPayload())));

        Flux<Object> result = gateway.publish("event");
```
_Example when published events have been modified by a dispatcher interceptor.
Such published modified events are returned to user as result Flux._


## Interceptors

Reactor gateway offers two types of interceptor: **dispatch** & result handler interceptor.

These interceptors allow us to centrally define rules & filter that will be applied to a message stream.

{% hint style="info" %}
Interceptors will be applied in order they have been registered.
{% endhint %}

### Dispatch Interceptors

Use this interceptor to centrally apply rules & validations for outgoing messages.

```java
        reactiveGateway
                .registerDispatchInterceptor(msgMono -> msgMono
                        .map(msg -> msg.andMetaData(Collections.singletonMap("key1", "value1"))));
```
_Dispatcher interceptor that adds key-value pair to message metadata._


```java
        reactiveGateway
                .registerDispatchInterceptor(msgMono -> msgMono
                        .filterWhen(v -> Mono.subscriberContext()
                                .filter(ctx-> ctx.hasKey("security"))
                                .map(ctx->ctx.get("security")))
                );
```
_Dispatcher Interceptor that discards the message if based on security flag in Reactor's Context._


### Result Handler Interceptors

Use this interceptor to centrally apply rules & validations for incoming messages (results).

Parameters are `message` that has been sent, and Flux of `results` for that message, that is going to be intercepted.

Message parameter can be useful if you want to apply result rule only for specific messages.

{% hint style="info" %}
This type of interceptor is available only in Reactor Extension.
{% endhint %}

```java
        reactiveGateway.registerResultHandlerInterceptor((msg, results) -> results
                .filter(r -> !r.getPayload().equals("blockedPayload")));
```
_Result interceptor that discards all results that have payload `blockedPayload`_


```java
        reactiveQueryGateway
                .registerResultHandlerInterceptor(
                        (query, results) -> results.flatMap(r -> {
                            if (r.getPayload().equals("")) {
                                return Flux.<ResultMessage<?>>error(new RuntimeException("no empty strings allowed"));
                            } else {
                                return Flux.just(r);
                            }
                        })
                );
```
_Result interceptor that validates that query result does not contain empty string._ 

```java
        reactiveQueryGateway
                .registerResultHandlerInterceptor((q, results) -> results
                        .filter(it -> !((boolean) q.getQueryName().equals("myBlockedQuery")))
                );
```
_Result interceptor that discards all results for `myBlockedQuery` query message._


```java
        reactiveGateway.registerResultHandlerInterceptor(
                (msg,results) -> results.timeout(Duration.ofSeconds(30)));
```
_Result interceptor that limits result waiting time to 30 seconds. (per message)_ 


```java
        reactiveGateway.registerResultHandlerInterceptor(
                (msg,results) -> results.log().take(5));
```
_Result interceptor that limits number of results to 5 entries, and logs all results._