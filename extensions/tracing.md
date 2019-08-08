# Axon Tracing

This extension provides functionality to trace command, event and query messages flowing through Axon application by providing a specific implementation of the `CommandGateway`, `QueryGateway`, `MessageDispatchInterceptor` and `MessageHandlerInterceptor`. 
The [Open Tracing](https://opentracing.io/) standard is used to provide the tracing capabilities, which thus allows usage of several Open Tracing implementations.

With this instrumentation, we can chain synchronous and asynchronous commands and queries, all belonging to the same parent span. A request can be visualized and analysed across Axon clients, command handlers, query handlers and event handlers, when running together or decomposed and deployed as separate parts (distributed).


```text
<dependency>
  <groupId>org.axonframework.extensions.tracing</groupId>
  <artifactId>axon-tracing-spring-boot-starter</artifactId>
  <version>4.1-M1</version>
</dependency>

<dependency>
  <groupId>io.opentracing.contrib</groupId>
  <artifactId>opentracing-spring-jaeger-web-starter</artifactId>
  <version>1.0.1</version>
</dependency>

```
The first dependency is [Spring Boot starter for Axon Tracing extension](../setting-up/maven-dependencies.md#axon-tracing-spring-boot-starter), which is the quickest start in to an extension configuration.

The second dependency is [Jaeger](https://www.jaegertracing.io/) implementation for OpenTracing.

There are other supported tracers that can be used: LightStep, Instana, Apache SkyWalking, Datadog, Wavefront by VMware, Elastic APM and many more.
