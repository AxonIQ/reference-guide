# Query processing

Axon Framework offers components for the query handling. 
Although, creating such a layer is fairly straight-forward, 
 using Axon Framework for this part of the application has a number of benefits, 
 such as the reuse of features such as interceptors and message monitoring.

The next sections provide an overview of the tasks related to setting up a query dispatching infrastructure with the Axon Framework.

Axon Framework makes a distinction between three types of queries:

1. [Point-to-Point queries](../implementing-domain-logic/query-handling/dispatching-queries.md#point-to-point-queries),
2. [Scatter-Gather queries](../implementing-domain-logic/query-handling/dispatching-queries.md#scatter-gather-queries), and
3. [Subscription queries](../implementing-domain-logic/query-handling/dispatching-queries.md#subscription-queries)

Click the links for specifics on how to implement each type of query.

## Query gateway

The query gateway is a convenient interface towards the query dispatching mechanism. 
While you are not required to use a gateway to dispatch queries,
 it is generally the easiest option to do so. 

Axon provides a `QueryGateway` interface and the `DefaultQueryGateway` implementation. 
The query gateway provides a number of methods that allow you to send a query and wait for a single or multiple results either synchronously,
 with a timeout or asynchronously. 
The query gateway needs to be configured with access to the query bus and a \(possibly empty\) list of `QueryDispatchInterceptor`s.

## Query bus

The query bus is the mechanism that dispatches queries to query handlers. 
Queries are registered using the combination of the query request name and query response type. 
It is possible to register multiple handlers for the same request-response combination,
 which can be used to implement for instance the scatter-gather pattern. 
When dispatching queries, the client must indicate whether it wants a response from a single handler or from all handlers.

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

The `SimpleQueryBus` does straightforward processing of queries in the thread that dispatches them. To configure `SimpleQueryBus` (instead of `AxonServerQueryBus`):

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
