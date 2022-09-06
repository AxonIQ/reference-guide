# Distributed Tracing

Distributed Tracing enables you to track the path of a message through your system to see how the system behaves and
performs.
When invoking multiple services, the context is propagated across each service so you can relate those calls.
It includes all related invocations across your microservice landscape.

Axon Framework supports both the OpenTelemetry and OpenTracing standards,
with OpenTelemetry being the default
since it has been adopted by the Cloud Native Foundation as the successor to OpenTracing.
While the OpenTracing standard only supported tracing, OpenTelemetry provides support for tracing, logs and metrics.

## OpenTelemetry <a id="opentelemetry"></a>

Axon work provides [OpenTelemetry support](https://opentelemetry.io/docs/concepts/what-is-opentelemetry/) out of the
box since version 4.6.0. The OpenTelemetry standard improves upon the OpenTracing standard by providing more
auto-instrumentation without the need
for the user to configure many things.

OpenTelemetry works by adding a Java agent to the execution of the application. Based on the configuration, the agent
will collect logs, metrics and tracing automatically before sending it to a collector that can provide insights.
ElasticSearch APM, Jaeger and many other tools are available for this and the configuration of these is beyond the scope
of this guide. You can find more
information [in the "Getting Started" section of the OpenTelemetry documentation.](https://opentelemetry.io/docs/instrumentation/java/getting-started/)

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

#### Spring boot

When using the Spring Boot autoconfiguration of Axon Framework,
everything will be autoconfigured when the `axon-tracing-opentelemetry` module is detected.

You might want to configure certain settings that are available.
The following table contains all configurable settings and their defaults.

| setting                                               | Default | Description                                                                                             |
|-------------------------------------------------------|---------|---------------------------------------------------------------------------------------------------------|
| `axon.tracing.showEventSourcingHandlers`              | `false` | Whether to show event sourcing handlers as a trace. This can be very noisy, and is thus off by default. |
| `axon.tracing.attributeProviders.aggregateIdentifier` | `true`  | Whether to add the aggregate identifier as a label when handling a message                              |
| `axon.tracing.attributeProviders.messageId`           | `true`  | Whether to add the message identifier as a label when handling a message                                |
| `axon.tracing.attributeProviders.messageName`         | `true`  | Whether to add the message name as a label when handling a message                                      |
| `axon.tracing.attributeProviders.messageType`         | `true`  | Whether to add the message type as a label when handling a message                                      |
| `axon.tracing.attributeProviders.payloadType`         | `true`  | Whether to add the payload type as a label when handling a message                                      |
| `axon.tracing.attributeProviders.metadata`            | `true`  | Whether to add the metadata properties as labels when handling a message                                |

In addition, you can customize the `OpenTelemetrySpanFactory` bean by defining it yourself.

```java
@Configuration
public class AxonConfiguration {

    @Bean
    public SpanFactory openTelemetrySpanFactory() {
        return OpenTelemetrySpanFactory().builder()
                                         // Customization can be done here
                                         .build();
    }
}
```

#### Manual configuration

The OpenTelemetry support can also be configured using the `Configurer` of Axon Framework to configure
the `OpenTelemetrySpanFactory`.

```java
Configurer configurer=DefaultConfigurer.defaultConfiguration()
        .configureSpanFactory(c->OpenTelemetrySpanFactory.builder().build());
```

Note that when not using Spring boot, tracing of each message handler invocation is not supported due to a limitation.

### Functionality

OpenTelemetry [supports a lot of libraries,
frameworks and application servers out of the box.](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/main/docs/supported-libraries.md)
For example, when a Spring REST endpoint is called it will automatically start a trace.
With the `axon-tracing-opentelemetry` module, this trace will be propagated to all subsequent Axon Framework messages.
For example, if the REST call produces a command which is sent over Axon Server,
handling the command will be included in the same trace as the original REST call.

In addition, the Axon Tracing OpenTelemetry module will trace the following functionality in Axon Framework:

- Dispatching and handling of commands
- Publishing of events
- Handling of events by event processors
- Dispatching and handling of queries
- Scheduling and invocation of deadlines
- Creation of snapshots
- Loading of the aggregate, including time spent waiting for a lock
- Axon Server GRPC calls
- Tracing of each message handler invocation (Spring Boot only)

When an action is synchronous, it will be part of the same trace. If an action is asynchronous,
such as creation of a snapshot after a command invocation is done, it will be a separate trace
linking back to the triggering trace.
This is true for event processors as well, linking back to the command that produced the event that is currently being
handled by the processor.

#### Span labels

Within OpenTelemetry, spans can be labeled with key-value pairs.
Axon Framework OpenTelemetry adds the following labels
to handling spans by default:

| label                       | description                                                                             |
|-----------------------------|:----------------------------------------------------------------------------------------|
| `axon_aggregate_identifier` | The aggregate identifier of the message, only present in case of a `DomainEventMessage` |
| `axon_message_id`           | The identifier of the message                                                           |
| `axon_message_name`         | The name of the message for Commands and Queries                                        |
| `axon_message_type`         | The class of the message, such as `DomainEventMessage` or `GenericQueryMessage`         |
| `axon_payload_type`         | The class of the payload in the message                                                 |
| `axon_metadata_{key}`       | All metadata of the message is also added to the span with its corresponding key        |

In addition to the ones provided by the framework, you can also create your own `SpanAttributesProvider`
and add it to the `SpanFactory` to add custom information you would like to include as a label.

```java
public class CustomSpanAttributesProvider implements SpanAttributesProvider {

    @Nonnull
    @Override
    public Map<String, String> provideForMessage(@Nonnull Message<?> message) {
        // Provide your own labels based on the message here
        return Collections.emptyMap();
    }
}
```

You can register this custom `SpanAttributesProvider` in several ways.

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
Configurer configurer=DefaultConfigurer.defaultConfiguration()
        .configureSpanFactory(c->OpenTelemetrySpanFactory.builder()
        .addSpanAttributeProviders(singletonList(new CustomSpanAttributesProvider()))
        .build());
```

{% endtab %}
{% endtabs %}

## OpenTracing <a id="opentracing"></a>

OpenTracing is still supported when you are not able to use OpenTelemetry.
For this, you can use [the OpenTracing extension to Axon Framework](../../extensions/tracing.md).
The functionality of the extensions is rahter limited compared to OpenTelemetry, so usage of OpenTelemetry is preferred.
