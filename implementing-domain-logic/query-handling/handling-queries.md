# Handling Queries

In Axon, an object may declare a number of query handler methods, by annotating them with `@QueryHandler`. The declared parameters of the method define which messages it will receive.

By default, `@QueryHandler` annotated methods allow the following parameter types:

* The first parameter is the payload of the query message. It may also be of type `Message` or `QueryMessage`, if the `@QueryHandler` annotation explicitly defined the name of the query the handler can process. By default, a query name is the fully qualified class name of the query its payload.
* Parameters annotated with `@MetaDataValue` will resolve to the metadata value with the key as indicated on the annotation. If `required` is `false` \(default\), `null` is passed when the metadata value is not present. If `required` is `true`, the resolver will not match and prevent the method from being invoked when the metadata value is not present.
* Parameters of type `MetaData` will have the entire metadata of a `QueryMessage` injected.
* Parameters of type `UnitOfWork` get the current unit of work injected. This allows query handlers to register actions to be performed at specific stages of the unit of work, or gain access to the resources registered with it.
* Parameters of type `Message`, or `QueryMessage` will get the complete message, with both the payload and the metadata. This is useful if a method needs several meta data fields, or other properties of the wrapping message.

You can configure additional `ParameterResolver`s by implementing the `ParameterResolverFactory` interface and creating a file named `/META-INF/service/org.axonframework.common.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class. See [Advanced customizations](../1.4-advanced-tuning/advanced-customizations.md) for details.

In all circumstances, at most one query handler method is invoked per query handling instance. Axon will search for the most specific method to invoke, using following rules:

1. On the actual instance level of the class hierarchy \(as returned by `this.getClass()`\), all annotated methods are evaluated
2. If one or more methods are found of which all parameters can be resolved to a value, the method with the most specific type is chosen and invoked
3. If no methods are found on this level of the class hierarchy, the super type is evaluated the same way
4. When the top level of the hierarchy is reached, and no suitable query handler is found, this query handling instance is ignored.

Note that similar to command handling, and unlike event handling, query handling does not take the class hierarchy of the query message into account.

```java
// assume QueryB extends QueryA 
// and    QueryC extends QueryB
// and    a single instance of SubHandler is registered

public class TopHandler {

    @QueryHandler
    public MyResult handle(QueryA query) {
    }

    @QueryHandler
    public MyResult handle(QueryB query) {
    }

    @QueryHandler
    public MyResult handle(QueryC query) {
    }
}

public class SubHandler extends TopHandler {

    @QueryHandler
    public MyResult handleEx(QueryB query) {
    }
}
```

In the example above, the handler method of `SubHandler` will be invoked for queries for `QueryB` and result `MyResult` the handler methods of `TopHandler` are invoked for queries for `QueryA` and `QueryC` and result `MyResult`.

## Registering Query Handlers

It is possible to register multiple query handlers for the same query name and type of response. When dispatching queries, the client can indicate whether he wants a result from one or from all available query handlers.

{% tabs %}
{% tab title="Axon Configuration API" %}
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
    .registerQueryHandler(conf -> new MyQueryHandler);
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
// Sample query handler
@Component
public class MyQueryHandler {
    @QueryHandler
    public String echo(String echo) {
        return echo;
    }
}
```
{% endtab %}
{% endtabs %}
