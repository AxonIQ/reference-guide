# Distributed Tracing

Distributed Tracing enables you to track the path of a message through your system to see how the system behaves and
performs.
Axon Framework provides additional tracing functionality to track what takes time in your microservice, such as how long
it took to load the aggregate, how long the actual command invocation took, or how long it took to publish events.

## Terminology

A **trace** is a collection of one or more **spans** that together form a complete journey through your software.
Creating a span that is not part of a trace will automatically create one with that span being the root span of the
trace.

Tools such as _ElasticSearch APM_ can render tracing information, as visible in the following image:

![Trace as shown in ElasticSearch APM when dispatching and handling a command.](/.gitbook/assets/tracing.png)

What we observe here is that a command is dispatched, distributed by Axon Server and handled. As a result of the command
an `AccountRegisteredEvent` is published and a deadline is scheduled as well.
In this image, the `AutomaticAccountCommandDispatcher.dispatch` span is the root trace, with each span being part of a
call hierarchy within that trace.

## Span factories

To provide additional insights in traces, many Axon Framework components use a `SpanFactory`.
This factory is responsible for the creation of multiple instances of a `Span` with a specific purpose.

You can use a `SpanFactory` provided the framework that matches your tracing standard.
Or, if your tracing standard of choice is not supported,
you can create one yourself by implementing the `SpanFactory` and `Span` interfaces.
The following standards are currently supported:

| Tracing standard | Supported | Description                                                                                                                                                                   |
|------------------|-----------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| OpenTelemetry    | Yes       | [OpenTelemetry](https://opentelemetry.io/docs/concepts/what-is-opentelemetry/) is the successor of OpenTracing, with auto-instrumentation being its most prominent feature.   |
| OpenTracing      | Limited   | [OpenTracing](https://opentracing.io/) is supported [by an extension with limited functionality](../../extensions/tracing.md). Usage of OpenTelemetry is recommended instead. |
| SLF4j            | Yes       | If you have no monitoring system in place but want to trace through logging, the framework provides a `LoggingSpanFactory`.                                                     |

You configure a `SpanFactory` in the following ways:

{% tabs %}
{% tab title="Spring Boot Bean" %}

```java
@Configuration
public class AxonConfiguration {

    @Bean
    public SpanFactory spanFactory() {
        // Any bean implementing the SpanFactory will be picked up automatically and override the defaults
        return new MyCustomSpanFactory();
    }
}
```

{% endtab %}
{% tab title="Axon Configurer" %}

```java
public class AxonConfigurer {

    public void configure(Configurer configurer) {
        configurer.configureSpanFactory(configuration -> new MyCustomSpanFactory());
    }
}
```

Note that this is not necessary for all providers, since some may provide Spring Boot autoconfiguration out of the box.
Refer to the specific subsections of this page on how to configure that specific `SpanFactory`.

{% endtab %}

### Combining factories

Sometimes you want the functionality of multiple `SpanFactory` implementations,
while Axon's configuration only allows one.
For this purpose, the framework contains the `MultiSpanFactory`,
which can be configured with multiple factories to which it delegates its calls.

For example, you can configure both the `LoggingSpanFactory` and the `OpenTelemetrySpanFactory` in the following
fashion:

{% tabs %}
{% tab title="Spring Boot Bean" %}

```java
@Configuration
public class AxonConfiguration {

    @Bean
    public SpanFactory spanFactory() {
        return new MultiSpanFactory(
                Arrays.asList(
                        LoggingSpanFactory.INSTANCE,
                        OpenTelemetrySpanFactory
                                .builder()
                                .build()
                )
        );
    }
}
```

{% endtab %}
{% tab title="Axon Configurer" %}

```java
public class AxonConfigurer {

    public void configure(Configurer configurer) {
        configurer.configureSpanFactory(configuration -> new MultiSpanFactory(
                Arrays.asList(
                        LoggingSpanFactory.INSTANCE,
                        OpenTelemetrySpanFactory.builder().build()
                )
        ));
    }
}
```

{% endtab %}

Whenever a span is now requested by the framework to be created, the factory calls a span that contains all spans of the
configured factories and acts like a single one.

## Features

The configured `SpanFactory` is responsible for creating spans when the framework requests it.
The framework specifies the type of span,
the name, and a message that triggered the span (if any, it's not required). The framework can request
the span types defined in the following table:

| Span Type     | Description                                                                                      | 
|---------------|--------------------------------------------------------------------------------------------------|
| Root trace    | Create a new trace entirely, having no parent.                                                   |
| Dispatch span | A span which is dispatching a message.                                                           |
| Handler span  | A span which is handling a message. Will set the span that dispatched the message as the parent. |
| Internal span | A span which specified something internal. It's not an entry or exit point.                     |

A trace generally consists of multiple spans with different types, depending on the functionality.
When a message is dispatched and handled by another process or thread, the handling trace is related to the dispatching
trace.
This handling trace becomes part of the root trace unless it's an asynchronous invocation. In that case it will be linked to it instead, 
so the tooling can refer to the related traces but the traces don't become so large they are unreadable.

The following functionality in Axon Framework is traced in addition to the tracing capabilities already provided by the
standard of your choice:

- Dispatching and handling of commands
- Publishing of events
- Handling of events by event processors
- Dispatching and handling of queries
- Scheduling and invocation of deadlines
- Creation of snapshots
- Loading of the aggregate, including time spent waiting for a lock
- Axon Server GRPC calls
- Tracing of each message handler invocation (Spring Boot only)

Tracing all of this functionality provided you with the best possible insight into the performance of your application.

### Span attribute providers

Most tracing implementations can add additional attributes to spans.
This is useful when debugging your application or finding a specific span you are looking for.
The framework provides the `SpanAttributesProvider`, which can be registered to the `SpanFactory` either via its
builder (if supported) or by calling the `SpanFactory.registerSpanAttributeProvider(provider)` method.

The following `SpanAttributesProvider` implementations are included in Axon Framework:

| Class                                       | label                       | description                                                                             |
|:--------------------------------------------|-----------------------------|:----------------------------------------------------------------------------------------|
| `AggregateIdentifierSpanAttributesProvider` | `axon_aggregate_identifier` | The aggregate identifier of the message, only present in case of a `DomainEventMessage` |
| `MessageIdSpanAttributesProvider`           | `axon_message_id`           | The identifier of the message                                                           |
| `MessageNameSpanAttributesProvider`         | `axon_message_name`         | The name of the message for Commands and Queries                                        |
| `MessageTypeSpanAttributesProvider`         | `axon_message_type`         | The class of the message, such as `DomainEventMessage` or `GenericQueryMessage`         |
| `PayloadTypeSpanAttributesProvider`         | `axon_payload_type`         | The class of the payload in the message                                                 |
| `MetadataSpanAttributesProvider`            | `axon_metadata_{key}`       | All metadata of the message is also added to the span with its corresponding key        |

In addition to the ones provided by the framework, you can also create a custom `SpanAttributesProvider`
and add it to the `SpanFactory`, in case you want to add custom information you would like to include as a label.

```java
public class CustomSpanAttributesProvider implements SpanAttributesProvider {

    @Nonnull
    @Override
    public Map<String, String> provideForMessage(@Nonnull Message<?> message) {
        // Provide your labels based on the message here
        return Collections.emptyMap();
    }
}
```

You can register this custom `SpanAttributesProvider` in one of the following ways.

{% tabs %}
{% tab title="Spring Boot Bean" %}

```java
@Configuration
public class AxonConfiguration {

    @Bean
    public SpanAttributesProvider customSpanAttributesProvider() {
        // Autoconfiguration picks beans of type SpanAttributesProvider up automatically.
        return new CustomSpanAttributesProvider();
    }
}
```

{% endtab %}
{% tab title="Spring Boot injection" %}

```java
@Configuration
public class AxonConfiguration {

    @Autowired
    public void configureSpanFactory(SpanFactory spanFactory) {
        spanFactory.registerSpanAttributeProvider(new CustomSpanAttributesProvider());
    }
}
```

{% endtab %}  
{% tab title="Manual" %}

```java
public class AxonConfigurer {

    public void configure(Configuration configuration) {
        configuration.spanFactory().registerSpanAttributeProvider(new CustomSpanAttributesProvider());
    }
}
```

{% endtab %}
{% endtabs %}

## OpenTelemetry <a id="opentelemetry"></a>

Axon work provides [OpenTelemetry support](https://opentelemetry.io/docs/concepts/what-is-opentelemetry/) out of the
box. The OpenTelemetry standard improves upon the OpenTracing standard by providing more
auto-instrumentation without the need for the user to configure many things.

OpenTelemetry works by adding a Java agent to the execution of the application. Based on the configuration, the agent
will collect logs, metrics and tracing automatically before sending it to a collector that can provide insights.
ElasticSearch APM, Jaeger and many other tools are available for this and the configuration of these is beyond the scope
of this guide. You can find more
information [in the "Getting Started" section of the OpenTelemetry documentation.](https://opentelemetry.io/docs/instrumentation/java/getting-started/)

OpenTelemetry [supports a lot of libraries,
frameworks and application servers out of the box.](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/main/docs/supported-libraries.md)
For example, when a Spring REST endpoint is called it will automatically start a trace.
With the `axon-tracing-opentelemetry` module, this trace will be propagated to all subsequent Axon Framework messages.
For example, if the REST call produces a command which is sent over Axon Server,
handling the command will be included in the same trace as the original REST call.

### Configuration

To get OpenTelemetry support enabled you will need to add the following dependency to your application's dependencies:

{% tabs %}
{% tab title="Maven" %}

```xml  
<dependency>  
    <groupId>org.axonframework</groupId>  
    <artifactId>axon-tracing-opentelemetry</artifactId>
	<version>${axon-framework.version}</version>
</dependency>
```

{% endtab %}%}
{% tab title="Gradle" %}

```gradle
implementation group: 'org.axonframework', name: 'axon-tracing-opentelemetry', version: axonFrameworkVersion
```

{% endtab %}
{% endtabs %}

Depending on your application, more configuration might be needed.

## Spring boot autoconfiguration

When using the Spring Boot autoconfiguration of Axon Framework, most things will be autoconfigured regardless of the
implementation.

You might want to configure certain settings that are available.
The following table contains all configurable settings, their defaults, and what they change:

| setting                                               | Default | Description                                                                                             |
|-------------------------------------------------------|---------|---------------------------------------------------------------------------------------------------------|
| `axon.tracing.showEventSourcingHandlers`              | `false` | Whether to show event sourcing handlers as a trace. This can be very noisy and is disabled by default. |
| `axon.tracing.attributeProviders.aggregateIdentifier` | `true`  | Whether to add the aggregate identifier as a label when handling a message                              |
| `axon.tracing.attributeProviders.messageId`           | `true`  | Whether to add the message identifier as a label when handling a message                                |
| `axon.tracing.attributeProviders.messageName`         | `true`  | Whether to add the message name as a label when handling a message                                      |
| `axon.tracing.attributeProviders.messageType`         | `true`  | Whether to add the message type as a label when handling a message                                      |
| `axon.tracing.attributeProviders.payloadType`         | `true`  | Whether to add the payload type as a label when handling a message                                      |
| `axon.tracing.attributeProviders.metadata`            | `true`  | Whether to add the metadata properties as labels when handling a message                                |

#### Manual configuration

The OpenTelemetry support can also be configured using the `Configurer` of Axon Framework to configure
the `OpenTelemetrySpanFactory`.

```java
public class AxonConfigurer {

    public void configure(Configurer configurer) {
        configurer.defaultConfiguration()
                  .configureSpanFactory(c -> OpenTelemetrySpanFactory.builder().build());
    }
}
```

Note that when not using Spring boot, tracing each message handler invocation is not supported due to a limitation.

## OpenTracing <a id="opentracing"></a>

OpenTracing is deprecated. If necessary, you can use [the OpenTracing extension to Axon Framework](../../extensions/tracing.md).
The functionality of the extension is rather limited compared to the OpenTelemetry integration, so using that if possible is preferred.

## Logging <a id="logging"></a>

Sometimes you don't have an APM system available, for instance during local development. It might still be useful to see
the traces that would be started and finished to obtain insights. For this purpose, the framework provides
a `LoggingSpanFactory`.

You can configure the `LoggingSpanFactory` in the following ways:
{% tabs %}
{% tab title="Spring Boot Bean" %}

```java
@Configuration
public class AxonConfiguration {

    @Bean
    public SpanFactory spanFactory() {
        return LoggingSpanFactory.INSTANCE;
    }
}
```

{% endtab %}
{% tab title="Manual" %}

```java
public class AxonConfigurer {

    public void configure(Configuration configuration) {
        configuration.configureSpanFactory(c -> LoggingSpanFactory.INSTANCE);
    }
}
```

{% endtab %}
{% endtabs %}
