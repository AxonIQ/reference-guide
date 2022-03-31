# Query Dispatchers

How to handle a query message has been covered in more detail in the [Query Handling section](query-handlers.md). Queries have to be dispatched, just like any type of message, before they can be handled. To that end Axon provides two interfaces:

1. The Query Bus, and
2. The Query Gateway

This page will show how and when to use the query gateway and bus. How to configure and specifics on the the query gateway and bus implementations are discussed [here](implementations.md)

## The Query Bus and Query Gateway

The `QueryBus` is the mechanism that dispatches queries to query handlers. Queries are registered using the combination of the query request name and query response type. It is possible to register multiple handlers for the same request-response combination, which can be used to implement the scatter-gather pattern. When dispatching queries, the client must indicate whether it wants a response from a single handler or from all handlers.

The `QueryGateway` is a convenient interface towards the query dispatching mechanism. While you are not required to use a gateway to dispatch queries, it is generally the easiest option to do so. It abstracts certain aspects for you, like the necessity to wrap a Query payload in a Query Message.

Regardless whether you choose to use the `QueryBus` or the `QueryGateway`, both provide several types of queries. Axon Framework makes a distinction between four types, being:

1. [Point-to-Point queries](query-dispatchers.md#point-to-point-queries),
2. [Scatter-Gather queries](query-dispatchers.md#scatter-gather-queries), 
3. [Subscription queries](query-dispatchers.md#subscription-queries) and
4. [Streaming queries](query-dispatchers.md#streaming-queries)

### Point-to-Point queries

The direct query represents a query request to a single query handler. If no handler is found for a given query, a `NoHandlerForQueryException` is thrown. In case multiple handlers are registered, it is up to the implementation of the Query Bus to decide which handler is actually invoked. In the listing below we have a simple query handler:

```java
@QueryHandler // 1.
public List<String> query(String criteria) {
    // return the query result based on given criteria
}
```

1. By default the name of the query is fully qualified class name of query payload \(`java.lang.String` in our case\).

   However, this behavior can be overridden by stating the `queryName` attribute of the `@QueryHandler` annotation.

If we want to query our view model, the `List<String>`, we would do something like this:

```java
// 1.
GenericQueryMessage<String, List<String>> query =
        new GenericQueryMessage<>("criteria", ResponseTypes.multipleInstancesOf(String.class));
// 2. send a query message and print query response
queryBus.query(query).thenAccept(System.out::println);
```

1. It is also possible to state the query name when we are building the query message,

   by default this is the fully qualified class name of the query payload.

2. The response of sending a query is a Java `CompletableFuture`,

   which depending on the type of the query bus may be resolved immediately.

   However, if a `@QueryHandler` annotated function's return type is `CompletableFuture`,

   the result will be returned asynchronously regardless of the type of the query bus.

### Scatter-Gather queries

When you want responses from all of the query handlers matching your query message, the scatter-gather query is the type to use. As a response to that query a stream of results is returned. This stream contains a result from each handler that successfully handled the query, in unspecified order. In case there are no handlers for the query, or all handlers threw an exception while handling the request, the stream is empty.

In the listing below we have two query handlers:

```java
@QueryHandler(queryName = "query")
public List<String> query1(String criteria) {
    // return the query result based on given criteria
}
```

```java
@QueryHandler(queryName = "query")
public List<String> query2(String criteria) {
    // return the query result based on given criteria
}
```

These query handlers could possibly be in different components and we would like to get results from both of them. So, we will use a scatter-gather query, like so:

```java
// create a query message
GenericQueryMessage<String, List<String>> query =
        new GenericQueryMessage<>("criteria", "query", ResponseTypes.multipleInstancesOf(String.class));
// send a query message and print query response
queryBus.scatterGather(query, 10, TimeUnit.SECONDS)
        .map(Message::getPayload)
        .flatMap(Collection::stream)
        .forEach(System.out::println);
```

### Subscription queries

The subscription query allows a client to get the initial state of the model it wants to query, and to stay up-to-date as the queried view model changes. In short it is an invocation of the Direct Query with the possibility to be updated when the initial state changes. To update a subscription with changes to the model, we will use the `QueryUpdateEmitter` component provided by Axon.

Let's take a look at a snippet from the `CardSummaryProjection`:

```java
@QueryHandler
public List<CardSummary> handle(FetchCardSummariesQuery query) {
    log.trace("handling {}", query);
    TypedQuery<CardSummary> jpaQuery = entityManager.createNamedQuery("CardSummary.fetch", CardSummary.class);
    jpaQuery.setParameter("idStartsWith", query.getFilter().getIdStartsWith());
    jpaQuery.setFirstResult(query.getOffset());
    jpaQuery.setMaxResults(query.getLimit());
    return log.exit(jpaQuery.getResultList());
}
```

This query handler will provide us with the list of GiftCard states. Once our GiftCard gets redeemed we would like to update any component which is interested in the updated state of that GiftCard. We'll achieve this by emitting an update using the `QueryUpdateEmitter` component within the event handler function of the `RedeemedEvt` event:

```java
@EventHandler
public void on(RedeemedEvt evt) {
    // 1.
    CardSummary summary = entityManager.find(CardSummary.class, event.getId());
    summary.setRemainingValue(summary.getRemainingValue() - event.getAmount());
    // 2.
    queryUpdateEmitter.emit(FetchCardSummariesQuery.class,
                            query -> event.getId().startsWith(query.getFilter().getIdStartsWith()),
                            summary);
}
```

1. First, we update our view model by updating the existing card.
2. If there is a subscription query interested in updates about this specific GiftCard we emit an update.

   The first parameter of the emission is the type of the query \(`FetchCardSummariesQuery` in our case\)

   which corresponds to the query type in a previously defined query handler.

   The second parameter is a predicate which will select the subscription query to be updated.

   In our case we will only update subscription queries interested in the GiftCard which has been updated.

   The third parameter is the actual update, which in our case is the card summary.

   There are several overloads of the emit method present, feel free to take a look at JavaDoc for more specifics on that.

   The important thing to underline here is that an update is a message and that some overloads take

   the update message as a parameter \(in our case we just sent the payload which was wrapped in the message\)

   which enables us to attach meta-data for example.

Once we have the query handling and the emitting side implemented, we can issue a subscription query to get the initial state of the GiftCard and be updated once this GiftCard is redeemed:

```java
// 1.
commandGateway.sendAndWait(new IssueCmd("gc1", amount)); 
// 2.
FetchCardSummariesQuery fetchCardSummariesQuery =
                new FetchCardSummariesQuery(offset, limit, filter);
// 3.
SubscriptionQueryResult<List<CardSummary>, CardSummary> fetchQueryResult = queryGateway.subscriptionQuery(
                fetchCardSummariesQuery,
                ResponseTypes.multipleInstancesOf(CardSummary.class),
                ResponseTypes.instanceOf(CardSummary.class));

fetchQueryResult
//4.
                .handle(cs -> cs.forEach(System.out::println), System.out::println)
//5.
                .doFinally(it -> fetchQueryResult.close());

// 6.
commandGateway.sendAndWait(new RedeemCmd("gc1", amount));
```

1. Issuing a GiftCard with `gc1` id and initial value of `amount`.
2. Creating a subscription query message to get the list of GiftCards

   \(this initial state is multiple instances of `CardSummary`\)

   and to be updated once the state of GiftCard with id `gc1` is changed \(in our case an update means the card is redeemed\).

   The type of the update is a single instance of `CardSummary`.

   Do note that the type of the update must match the type of the emission side.

3. Once the message is created, we are sending it via the `QueryGateway`.

   We receive a query result which contains two components: one is `initialResult` and the other is `updates`.

   In order to achieve 'reactiveness' we use [Project Reactor](https://projectreactor.io/)'s `Mono` for `initialResult`

   and `Flux` for `updates`.

> **Note**
>
> Once the subscription query is issued, all updates are queued until the subscription to the `Flux` of `updates` is done. This behavior prevents the losing of updates.
>
> **Note**
>
> The Framework prevents issuing more than one query message with the same id. If it is necessary to be updated in several different places, create a new query message.
>
> **Note**
>
> The `reactor-core` dependency is mandatory for usage of subscription queries. However, it is a compile time dependency and it is not required for other Axon features.

1. The `SubscriptionQueryResult#handle(Consumer<? super I>, Consumer<? super U>)`

   method gives us the possibility to subscribe to the `initialResult` and the `updates` in one go.

   If we want more granular control over the results, we can use the `initialResult()` and `updates()` methods on the query result.

2. As the `queryUpdateEmitter` will continue to emit updates even when there are no subscribers, we need to notify the emitting side once we are no longer interested in receiving updates.

   Failing to do so can result in hanging infinitive streams and eventually a memory leak.

   Once we are done with using subscription query, we need to close the used resource. We can do that in `doFinally` hook.

   As an alternative to the `doFinally` hook, there is the `Flux#using` API. This is synonymous

   to the try-with-resource Java API:

   ```text
   Flux.using( () -> fetchQueryResult, 
            queryResult -> queryResult.handle(..., ...), 
            SubscriptionQueryResult::close
        );
   ```

3. When we issue a `RedeemCmd`, our event handler in the projection will eventually be triggered,

   which will result in the emission of an update.

   Since we subscribed to updates with the `println()` method, the update will be printed out once it is received.


### Streaming queries

The streaming query allows a client to, for example, stream large database result sets. The streaming query relies on the reactive stream model, specifically the `Flux` type.


The streaming query is flexible enough to handle **any** query return type. That means that any return type that is not a `Flux` will automatically be converted to `Flux`. The `Flux` will emit one or multiple items based on query handler.

The `QueryGateway` provides the `streamingQuery` method to utilize the streaming query. 
It's simple to use and requires just two parameters: the query payload and the expected response type class.
Note that the `streamingQuery` method **is lazy**, meaning the query is sent once the `Flux` is subscribed to.

```java
@QueryHandler
public List<CardSummary> handle(FetchCardSummariesQuery query) {
        ...
    return cardRepository.findAll(); //1
}
        ...

public Flux<CardSummary> consumer() {
        return queryGateway.streamingQuery(query, CardSummary.class); //2
}
```

1. We are querying the `cardRepository` for all the cards, that can potently return a large result set containing thousands of items.
2. We are using the `queryGateway` to issue the query. If we used `multipleInstanceOf(CardSummary.class)` we would get large list transferred as single result message over the network, potentially causing a buffer overflow and max message size violation.
Instead, we are using `streamingQuery(query, CardSummary.class)`, that will convert our response to Flux stream and chunk the result set into smaller messages of CardSummary.

Natively, if we want fine-grained control of the producing stream we can use Flux as return type:

```java
@QueryHandler
public Flux<CardSummary> handle(FetchCardSummariesQuery query) {
        ...
    return reactiveCardRepository.findAll(); 
}
```

When using Flux as return type, we can control backpressure, stream cancellation and implement more complex features like pagination.

> **Transaction Leaking Concerns**
>
> Once a consumer of the streaming query receives the `Flux` to subscribe to, the transaction will be considered completed successfully. 
> That means that any subsequent messages on the stream will not be part of the transaction, including errors.
> As the transaction is already over an error will not be propagated to the parent transaction to invoke any rollback method.
> This has the implication that the streaming query should not be used within a Unit Of Work (within message handlers or any other transactional methods) to chain other transactional actions (like sending a command or query).

#### Streaming back-pressure

Back-pressure (flow control) is an essential feature in reactive systems that allows consumers to control the data flow, ensuring they are not overwhelmed by the producer.
The streaming query implements a pull-based back-pressure strategy, which means that the producer will emit data when the consumer is ready to receive it.

If you are using Axon Server, for more information see the [flow control documentation](../../axon-server/performance/flow-control.md).

#### Cancellation

The streaming query can be implemented as an infinitive stream.
Hence, it's important to cancel it once the client is not interested in receiving any more data.

The following sample shows how this could be achieved:


```java
public Flux<CardSummary> consumer() {
        return queryGateway.streamingQuery(query, CardSummary.class)
                           .take(100)
                           .takeUntil(message -> somePredicate.test(message));
}
\```

The example above shows how the `take` operator limits the number of items to be emitted.


#### Error handling

Producer that produces an error by calling `onError(Throwable)` will terminate the handler execution and the consumer will have its `onError(Throwable)` subscription handler called as expected.

Exceptions do not flow upstream (from consumer to producer). 
If an error happens on consumer side, consumer error will trigger cancel signal that will be propagated to producer, and effectively cancel the stream, without producer knowing the reason.

It's recommended to set a timeout on a query handler side in case of finite stream to protect against malfunctioning consumer or producer.

```java
@QueryHandler
public Flux<CardSummary> handle(FetchCardSummariesQuery query) {
...
return reactiveCardRepository.findAll().timeout(Duration.ofSeconds(5));
}
```
Example above shows usage of `timeout` operator to cancel a request of response has not observed during 5 second span.



**Can streaming query replace subscription query?**

Depends on your setup, but most likely not. 
First, we would need assume that projection side uses compliant reactive db driver.
Second, not all db drivers can stream updates for table changes out of the box.
For example, PostgreSQL support LISTEN and NOTIFY features, and MongoDB supports tailorable cursor, but this is something that client needs to set up himself.
Subscription queries are still important feature of Axon Framework that has no alternative when used with traditional db drivers.

> **Reactor dependecy**
>
> The `reactor-core` dependency is mandatory for usage of streaming queries. However, it is a compile time dependency and it is not required for other Axon features.


[Axon Coding Tutorial \#5: - Connecting the UI](https://youtu.be/lxonQnu1txQ)

