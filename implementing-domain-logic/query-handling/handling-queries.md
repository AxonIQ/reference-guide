# Handling Queries

The handling of a query comes down to an annotated handler returning the queries response. 
The goal of this chapter is to describe what such an `@QueryHandler` annotated method looks like,
 as well as describing the call order and response type options.
For configuration of query handlers and the `QueryBus`,
 it is recommended to read the [Query Processing](../../configuring-infrastructure-components/query-processing/query-processing.md) section.

## Writing a Query Handler

In Axon, an object may declare a number of query handler methods,
 by annotating them with the `@QueryHandler` annotation.
The object in question is what you would refer to as the Query Handler, or Query Handling Component.
For a query handler method, the first declared parameter defines which query message object it will receive.

Taking the 'Gift Card' domain which contains a `CardSummary` Query Model,
 we can assume there is a query message to fetch a single `CardSummary` instance.
Let us define the format of the query message as follows:

```java
public class FetchCardSummaryQuery {
    
    private final String cardSummaryId;
    
    public FetchCardSummaryQuery(String cardSummaryId) {
        this.cardSummaryId = cardSummaryId;
    }
    // omitted getters, equals/hashCode, toString functions
}
```

As shown, we have a regular POJO that will fetch a `CardSummary` based on the `cardSummaryId` field.
This `FetchCardSummaryQuery` will be [dispatched](dispatching-queries.md) to a handler that defines the given message as
 its first declared parameter.
The handler will likely be contained in an object
 which is in charge of or has access to the `CardSummary` model in question:

```java
import org.axonframework.queryhandling.QueryHandler;

public class CardSummaryProjection {
    
    private Map<String, CardSummary> cardSummaryStorage;
   
    @QueryHandler // 1.
    public CardSummary handle(FetchCardSummaryQuery query) { // 2.
        return cardSummaryStorage.get(query.getCardSummaryId());
    }
    // omitted CardSummary event handlers which update the model
}
```

From the above sample we want to highlight two specifics when it comes to writing a query handler:

 1. The `@QueryHandler` annotation which marks a function as a query handler method.
 2. The method in question is defined by the return type `CardSummary`, which is called the query response type, and the `FetchCardSummaryQuery` which is the query payload.
 
> **Storing a Query Model**
>
> For the purpose of the example we have chosen to use a regular `Map` as the storage approach.
> In a real life system, this would be replaced by a form of database or repository layer for example.

## Query Handler Call Order

In all circumstances, at most one query handler method is invoked per query handling instance. 
Axon will search for the most specific method to invoke, using following rules:

1. On the actual instance level of the class hierarchy \(as returned by `this.getClass()`\), all annotated methods are evaluated
2. If one or more methods are found of which all parameters can be resolved to a value, the method with the most specific type is chosen and invoked
3. If no methods are found on this level of the class hierarchy, the super type is evaluated the same way
4. When the top level of the hierarchy is reached, and no suitable query handler is found, this query handling instance is ignored.

Note that similar to command handling, and unlike event handling,
 query handling does not take the class hierarchy of the query message into account.

```java
// assume QueryB extends QueryA 
// and    QueryC extends QueryB
// and    a single instance of SubHandler is registered

public class QueryHandler {

    @QueryHandler
    public MyResult handle(QueryA query) {
        // Return result
    }

    @QueryHandler
    public MyResult handle(QueryB query) {
        // Return result
    }

    @QueryHandler
    public MyResult handle(QueryC query) {
        // Return result
    
    }
}

public class SubQueryHandler extends QueryHandler {

    @QueryHandler
    public MyResult handleEx(QueryB query) {
        // Return result
    }
}
```

In the example above, the handler method of `SubQueryHandler` will be invoked for queries for `QueryB`
 and result `MyResult` the handler methods of `QueryHandler` are invoked for queries for `QueryA` and `QueryC` 
 and result `MyResult`.
