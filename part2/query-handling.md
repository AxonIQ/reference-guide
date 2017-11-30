Query Handling
==============

Query handling components act on incoming query messages. They typically read data from the view models created by the Event listeners. Query handling components typically do not raise new Events or send Commands.

Defining Query Handlers
=======================

In Axon, an object may declare a number of Query Handler methods, by annotating them with `@QueryHandler`. The declared parameters of the method define which messages it will receive.

By default, `@QueryHandler` annotated methods allow the following parameter types:

- The first parameter is the payload of the Query Message. It may also be of type `Message` or `QueryMessage`, if the `@QueryHandler` annotation explicitly defined the name of the Query the handler can process. By default, a Query name is the fully qualified class name of the Query's payload.

- Parameters annotated with `@MetaDataValue` will resolve to the Meta Data value with the key as indicated on the annotation. If `required` is `false` (default), `null` is passed when the meta data value is not present. If `required` is `true`, the resolver will not match and prevent the method from being invoked when the meta data value is not present.

- Parameters of type `MetaData` will have the entire `MetaData` of a `QueryMessage` injected.

- Parameters of type `UnitOfWork` get the current Unit of Work injected. This allows query handlers to register actions to be performed at specific stages of the Unit of Work, or gain access to the resources registered with it.

- Parameters of type `Message`, or `QueryMessage` will get the complete message, with both the payload and the Meta Data. This is useful if a method needs several meta data fields, or other properties of the wrapping Message.

Registering Query Handlers
==========================

It is possible to register multiple query handlers for the same query name and type of response. When dispatching queries, the client can indicate whether he wants a result from one or from all available query handlers.

Using Spring
------------
When using Spring AutoConfiguration, all singleton Spring beans are scanned for methods that have the `@QueryHandler` annotation. For each method that is found, a new query handler is registered with the query bus.

Using Configuration API
-----------------------
It is also possible to use the Configuration API to register query handlers. To do so, use the `registerQueryHandler` method on the `Configurer` class:

```java
// Sample query handler
public class MyQueryHandler {
    @QueryHandler
    public String echo(String echo) {
        return echo;
    }
}

...

// To register your query handler
Configurer axonConfigurer = DefaultConfigurer.defaultConfiguration()
    .registerQueryHandler((conf) -> new MyQueryHandler);
```
