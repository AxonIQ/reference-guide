Query Dispatching
=================
Since version 3.1 Axon Framework also offers components for the Query handling. Although creating such a layer is fairly straight-forward, using Axon Framework for this part of the application has a number of benefits, such as the reuse of features such as interceptors and message monitoring. 

The next sections provide an overview of the tasks related to setting up a Query dispatching infrastructure with the Axon Framework.

Query Gateway
=============
The Query Gateway is a convenient interface towards the Query dispatching mechanism. While you are not required to use a Gateway to dispatch Queries, it is generally the easiest option to do so. Axon provides a `QueryGateway` interface and the `DefaultQueryGateway` implementation. The query gateway provides a number of methods that allow you to send a query and wait for a single or multiple results either synchronously, with a timeout or asynchronously. The query gateway needs to be configured with access to the Query Bus and a (possibly empty) list of `QueryDispatchInterceptor`s.

Query Bus
=========
The Query Bus is the mechanism that dispatches queries to Query Handlers. Queries are registered using the combination of the query request name and query response type. It is possible to register multiple handlers for the same request-response combination, which can be used to implement for instance the scatter-gather pattern. When dispatching queries, the client must indicate whether it wants a response from a single handler or from all handlers.

If the client requests a response from a single handler, and no handler is found, a `NoHandlerForQueryException` is thrown. In case multiple handlers are registered, it is up to the implementation of the Query Bus to decide which handler is actually invoked.

If the client requests a response from all handlers, a stream of results is returned. This stream contains a result from each handler that successfully handled the query, in unspecified order. In case there are no handlers for the query, or all handlers threw an exception while handling the request, the stream is empty.

SimpleQueryBus
--------------
The `SimpleQueryBus` is the only Query Bus implementation in Axon 3.1. It does straightforward processing of queries in the thread that dispatches them. The `SimpleQueryBus` allows interceptors to be configured, see [Query Interceptors](#query-interceptors) for more information.

Query Interceptors
==================
One of the advantages of using a query bus is the ability to undertake action based on all incoming queries. Examples are logging or authentication, which you might want to do regardless of the type of query. This is done using Interceptors.

There are different types of interceptors: Dispatch Interceptors and Handler Interceptors. Dispatch Interceptors are invoked before a query is dispatched to a Query Handler. At that point, it may not even be sure that any handler exists for that query. Handler Interceptors are invoked just before a Query Handler is invoked.

Dispatch Interceptors
---------------------
Message Dispatch Interceptors are invoked when a query is dispatched on a Query Bus. They have the ability to alter the Query Message, by adding Meta Data, for example, or block the query by throwing an Exception. These interceptors are always invoked on the thread that dispatches the Query.
Message Dispatch Interceptors are invoked when a query is dispatched on a Query Bus. They have the ability to alter the Query Message, by adding Meta Data, for example, or block the query by throwing an Exception. These interceptors are always invoked on the thread that dispatches the Query.

### Structural validation

There is no point in processing a query if it does not contain all required information in the correct format. In fact, a query that lacks information should be blocked as early as possible. Therefore, an interceptor should check all incoming queries for the availability of such information. This is called structural validation.

Axon Framework has support for JSR 303 Bean Validation based validation. This allows you to annotate the fields on queries with annotations like `@NotEmpty` and `@Pattern`. You need to include a JSR 303 implementation (such as Hibernate-Validator) on your classpath. Then, configure a `BeanValidationInterceptor` on your Query Bus, and it will automatically find and configure your validator implementation. While it uses sensible defaults, you can fine-tune it to your specific needs.

> **Tip**
>
> You want to spend as few resources on an invalid queries as possible. Therefore, this interceptor is generally placed in the very front of the interceptor chain. In some cases, a Logging or Auditing interceptor might need to be placed in front, with the validating interceptor immediately following it.

The BeanValidationInterceptor also implements `MessageHandlerInterceptor`, allowing you to configure it as a Handler Interceptor as well.

Handler Interceptors
--------------------
Message Handler Interceptors can take action both before and after query processing. Interceptors can even block query processing altogether, for example for security reasons.

Interceptors must implement the `MessageHandlerInterceptor` interface. This interface declares one method, `handle`, that takes three parameters: the query message, the current `UnitOfWork` and an `InterceptorChain`. The `InterceptorChain` is used to continue the dispatching process.

Unlike Dispatch Interceptors, Handler Interceptors are invoked in the context of the Query Handler. That means they can attach correlation data based on the Message being handled to the Unit of Work, for example. This correlation data will then be attached to messages being created in the context of that Unit of Work.