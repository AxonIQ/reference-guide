# Query processing

Axon Framework offers components for the query handling. Although, creating such a layer is fairly straight-forward, using Axon Framework for this part of the application has a number of benefits, such as the reuse of features such as interceptors and message monitoring.

The next sections provide an overview of the tasks related to setting up a query dispatching infrastructure with the Axon Framework.

## Query gateway

The query gateway is a convenient interface towards the query dispatching mechanism. While you are not required to use a gateway to dispatch queries, it is generally the easiest option to do so. Axon provides a `QueryGateway` interface and the `DefaultQueryGateway` implementation. The query gateway provides a number of methods that allow you to send a query and wait for a single or multiple results either synchronously, with a timeout or asynchronously. The query gateway needs to be configured with access to the query bus and a \(possibly empty\) list of `QueryDispatchInterceptor`s.

## Query bus

The query bus is the mechanism that dispatches queries to query handlers. Queries are registered using the combination of the query request name and query response type. It is possible to register multiple handlers for the same request-response combination, which can be used to implement for instance the scatter-gather pattern. When dispatching queries, the client must indicate whether it wants a response from a single handler or from all handlers.

The Query Bus is the mechanism that dispatches queries to Query Handlers. Queries are registered using the combination of the query request name and query response type. It is possible to register multiple handlers for the same request-response combination. Axon supports 3 query types:

* Direct query
* Scatter-gather query
* Subscription query

### Direct query

The direct query represents a query request to a single query handler. If no handler is found for a given query, a `NoHandlerForQueryException` is thrown. In case multiple handlers are registered, it is up to the implementation of the Query Bus to decide which handler is actually invoked. In the listing below we have a simple query handler:

```java
@QueryHandler // (1)
public List<String> query(String criteria) {
    // return the query result based on given criteria
}
```

\(1\) By default the name of the query is fully qualified class name of query payload \(`java.lang.String` in our case\). However, this behavior can be overridden by stating the `queryName` attribute of the `@QueryHandler` annotation.

If we want to query our view model, the `List<String>`, we would do something like this:

```java
// (1) create a query message
GenericQueryMessage<String, List<String>> query =
        new GenericQueryMessage<>("criteria", ResponseTypes.multipleInstancesOf(String.class));
// (2) send a query message and print query response
queryBus.query(query).thenAccept(System.out::println);
```

\(1\) It is also possible to state the query name when we are building the query message, by default this is the fully qualified class name of the query payload.

\(2\) The response of sending a query is a java `CompletableFuture`, which depending on the type of the query bus may be resolved immediately. However, if a `@QueryHandler` annotated function's return type is `CompletableFuture`, the result will be returned asynchronously regardless of the type of the query bus.

### Scatter-gather query

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

Since they are query handlers that are possibly in different components we would like to get result from both of them. So, we will use a scatter-gather query, like so:

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

### Subscription query

The subscription query allows a client to get the initial state of the model it wants to query, and to stay up-to-date as the queried view model changes. In short it is an invocation of the Direct Query with possibility to be updated when the initial state changes. To update a subscription with changes to the model, we will use the `QueryUpdateEmitter` component provided by Axon.

Let's take a look at `CardSummaryProjection` in the [Getting Started](../1-axon-framework/getting-started.md) section:

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
    // (1)
    CardSummary summary = entityManager.find(CardSummary.class, event.getId());
    summary.setRemainingValue(summary.getRemainingValue() - event.getAmount());
    // (2)
    queryUpdateEmitter.emit(FetchCardSummariesQuery.class,
                            query -> event.getId().startsWith(query.getFilter().getIdStartsWith()),
                            summary);
}
```

\(1\) First, we update our view model by updating the existing card.

\(2\) If there is a subscription query interested in updates about this specific GiftCard we emit an update. The first parameter of the emission is the type of the query \(`FetchCardSummariesQuery` in our case\) which corresponds to the query type in previously defined query handler. The second parameter is a predicate which will select the subscription query to be updated. In our case we will update only subscription queries interested in the GiftCard which has been updated. The third parameter is the actual update, which in our case is the card summary. There are several overloads of the emit method present, feel free to take a look at JavaDoc for more specifics on that. Important thing to underline here is that an update is a message and that some overloads take the update message as a parameter \(in our case we just sent the payload which was wrapped in the message\) which enables us to attach meta-data for example.

Once we have query handling and emitting side implemented, we can issue a subscription query to get initial state of the GiftCard and be updated once this GiftCard is redeemed:

```java
// (1)
commandGateway.sendAndWait(new IssueCmd("gc1", amount)); 
// (2)
FetchCardSummariesQuery fetchCardSummariesQuery =
                new FetchCardSummariesQuery(offset, limit, filter);
// (3)
SubscriptionQueryResult<List<CardSummary>, CardSummary> fetchQueryResult = queryGateway.subscriptionQuery(
                fetchCardSummariesQuery,
                ResponseTypes.multipleInstancesOf(CardSummary.class),
                ResponseTypes.instanceOf(CardSummary.class));
// (4)
fetchQueryResult.handle(cs -> cs.forEach(System.out::println), System.out::println);
// (5)
commandGateway.sendAndWait(new RedeemCmd("gc1", amount));
```

\(1\) Issuing a GiftCard with `gc1` id and initial value of `amount`.

\(2\) Creating a subscription query message to get the list of GiftCards \(this initial state is multiple instances of `CardSummary`\) and to be updated once the state of GiftCard with id `gc1` is changed \(in our case update means the card is redeemed\). The type of the update is a single instance of `CardSummary`. Do note that the type of the update must match the type of the emission side.

\(3\) Once the message is created, we are sending it via the `QueryGateway`. We receive a query result which contains two components: one is `initialResult` and the other is `updates`. In order to achieve 'reactiveness' we use [Project Reactor](https://projectreactor.io/)'s `Mono` for `initialResult` and `Flux` for `updates`.

> **Note** Once the subscription query is issued, all updates are queued until the subscription to the `Flux` of `updates` is done. This behavior prevents losing of updates.
>
> **Note** The Framework prevents issuing more than one query message with the same id. If it is necessary to be updated in several different places, create a new query message.
>
> **Note** `reactor-core` dependency is mandatory for usage of subscription queries. However, it is a compile time dependency and it is not required for other Axon features.

\(4\) The `SubscriptionQueryResult#handle(Consumer<? super I>, Consumer<? super U>)` method gives us the possibility to subscribe to the `initialResult` and the `updates` in one go. If we want more granular control over the results, we can use the `initialResult()` and `updates()` methods on the query result.

\(5\) When we issue a `RedeemCmd`, our event handler in the projection will eventually be triggered, which will result in the emission of an update. Since we subscribed with the `println()` method to updates, the update will be printed out once it is received.


### AxonServerQueryBus

Axon provides a query bus out of the box, the `AxonServerQueryBus`. It connects to the [AxonIQ AxonServer Server](/introduction/axon-server.md) to send and receive Queries.

`AxonServerQueryBus` is a 'distributed query bus'. It is using [`SimpleQueryBus`](query-processing.md#simplequerybus) to handle incoming queries on different JVM's by default.

{% tabs %}
{% tab title="Axon Configuration API" %}

Declare dependencies:
```
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-server-connector</artifactId>
    <version>${axon.version}</version>
</dependency>
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-configuration</artifactId>
    <version>${axon.version}</version>
</dependency>
```
Configure your application:
```java
// Returns a Configurer instance with default components configured. `AxonServerQueryBus` is configured as Query Bus by default.
Configurer configurer = DefaultConfigurer.defaultConfiguration();
```
> NOTE: If you exclude `axon-server-connector` dependency you will fallback to 'non-axon-server' query bus option: `SimpleQueryBus` (see [below](query-processing.md#simplequerybus))

{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}

By simply declaring dependency to `axon-spring-boot-starter`, Axon will automatically configure the Axon Server Query Bus:
```
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-spring-boot-starter</artifactId>
    <version>${axon.version}</version>
</dependency>

```
{% endtab %}
{% endtabs %}

### SimpleQueryBus

The `SimpleQueryBus` does straightforward processing of queries in the thread that dispatches them. The `SimpleQueryBus` allows interceptors to be configured.

{% tabs %}
{% tab title="Axon Configuration API" %}

```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
            .configureQueryBus(c -> SimpleQueryBus.builder().transactionManager(c.getComponent(TransactionManager.class)).messageMonitor(c.messageMonitor(SimpleQueryBus.class, "queryBus")).build());
 ```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Bean
public SimpleQueryBus queryBus(AxonConfiguration axonConfiguration, TransactionManager transactionManager) {
    return SimpleQueryBus.builder()
                         .messageMonitor(axonConfiguration.messageMonitor(QueryBus.class, "queryBus"))
                         .transactionManager(transactionManager)
                         .errorHandler(axonConfiguration.getComponent(
                                 QueryInvocationErrorHandler.class,
                                 () -> LoggingQueryInvocationErrorHandler.builder().build()
                         ))
                         .queryUpdateEmitter(axonConfiguration.getComponent(QueryUpdateEmitter.class))
                         .build();
}

```
> NOTE: If you exclude `axon-server-connector` dependency from `axon-spring-boot-starter` you will have `SimpleQueryBus` component auto-configured and loaded into Spring Application Context by default, and you don't need to explicitly define this component in you Spring configuration.

{% endtab %}
{% endtabs %}
