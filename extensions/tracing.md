# Tracing

This extension provides functionality to trace command, event and query messages flowing through an Axon application by providing a specific implementation of the `CommandGateway`, `QueryGateway`, `MessageDispatchInterceptor` and `MessageHandlerInterceptor`. The [Open Tracing](https://opentracing.io/) standard is used to provide tracing capabilities, which thus allows usage of several Open Tracing implementations.

With this instrumentation, we can chain synchronous and asynchronous commands and queries, all belonging to the same parent span. A request can be visualized and analysed across Axon clients, command handlers, query handlers and event handlers, when running together or decomposed and deployed as separate parts \(distributed\).

```text
<dependency>
  <groupId>org.axonframework.extensions.tracing</groupId>
  <artifactId>axon-tracing-spring-boot-starter</artifactId>
  <version>4.4</version>
</dependency>

<dependency>
  <groupId>io.opentracing.contrib</groupId>
  <artifactId>opentracing-spring-jaeger-web-starter</artifactId>
  <version>3.2.2</version>
</dependency>
```

The first dependency is [Spring Boot starter for Axon Tracing extension](../axon-framework/modules.md#axon-tracing-spring-boot-starter), which is the quickest start in to an extension configuration.

The second dependency is [Jaeger](https://www.jaegertracing.io/) implementation for OpenTracing.

There are other supported tracers that can be used: LightStep, Instana, Apache SkyWalking, Datadog, Wavefront by VMware, Elastic APM and many more.

## Configuring the extension
The extension can be disabled setting the property `axon.extension.tracing.enabled` to `false` (default=`true`). This will give you the possibility to turn it off when needed (e.g.: for a certain environment).

Furthermore there is a more fine-grained configuration option of the tracing span tags on commands, events and queries.  You can customize span tags easily, mixing and matching between available tag `messageId`, `aggregateId`, `messageType`, `payloadType`, `messageName` and `payload`. Take into account that some of the tags make sense on a certain span type, but not on another, and some of them have an hidden cost on network (such as payload). Use them wisely!
```
axon.extension.tracing.span.commandTags=messageId, messageType, payloadType, messageName
axon.extension.tracing.span.eventTags=messageId, aggregateId, messageType, payloadType
axon.extension.tracing.span.queryTags=messageId, messageType, payloadType, messageName
```

Above an example of the default value.
Available tags field are listed in [MessageTag.java](https://github.com/AxonFramework/extension-tracing/blob/master/tracing/src/main/java/org/axonframework/extensions/tracing/MessageTag.java) class.
