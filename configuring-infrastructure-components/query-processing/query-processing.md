# Query Processing

Processing [queries](../messaging-concepts/messaging-concepts.md#queries) as a dedicated type of message is in line with
 segregating out query models when following [CQRS](../../introduction/architecture-overview/ddd-cqrs-concepts.md). 
Although creating a query handling layer is fairly straight-forward, 
 using Axon Framework for this part of the application has a number of benefits.
 
Through providing the functionality to describe query handling methods
 (as further explained in [this](../../implementing-domain-logic/query-handling/query-handling.md) section) 
 and a dedicated bus for query messages, 
 common message features such as [interceptors](../messaging-concepts/message-intercepting.md) and
  [message monitoring](../../operations-guide/production-considerations/monitoring-and-metrics.md) can used.

The next sections provide an overview of the tasks related to configuring the necessary components to start processing
 queries in an Axon application.
To that end the approach to registering `@QueryHandler` annotated methods is discussed,
 as well as what options are present when it comes to dispatching queries.

> **Query Types**
> 
> Axon Framework makes a distinction between three types of queries:
>  1. [Point-to-Point queries](../../implementing-domain-logic/query-handling/dispatching-queries.md#point-to-point-queries),
>  2. [Scatter-Gather queries](../../implementing-domain-logic/query-handling/dispatching-queries.md#scatter-gather-queries), and
>  3. [Subscription queries](../../implementing-domain-logic/query-handling/dispatching-queries.md#subscription-queries)
> 
> Click the links for specifics on how to implement each type of query.
