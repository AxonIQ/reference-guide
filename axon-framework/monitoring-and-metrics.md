# Monitoring and Metrics

The ability to monitor and measure what is going on is very important. Especially in a location transparent environment like an Axon application it is very important to be able to trace your message and check the ingestion rate of it.‌

## Monitoring <a id="monitoring"></a>

‌

Monitoring a message centric application will require you to be able to see where your messages are at a given point in time. This translates to being able to track your commands, events and queries from one component to another in an Axon application.‌

### Correlation Data <a id="correlation-data"></a>

One import aspect in regards to this is tracing a given message. To that end the framework provides the `CorrelationDataProvider`, as described briefly [here](messaging-concepts/message-intercepting.md). This interface and its implementations provide you the means to populate the meta-data of your messages with specific fields, like a 'trace-id', 'correlation-id' or any other field you might be interested in.‌

For configuring the `MessageOriginProvider` you can do the following:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class MonitoringConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    public void configureMessageOriginProvider(Configurer configurer) {
        configurer.configureCorrelationDataProviders(configuration -> {
            List<CorrelationDataProvider> correlationDataProviders = new ArrayList<>();
            correlationDataProviders.add(new MessageOriginProvider());
            return correlationDataProviders;
        });
    }

}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```text
public class MonitoringConfiguration {

    // When using Spring Boot, simply defining a CorrelationDataProvider bean is sufficient
    public CorrelationDataProvider messageOriginProvider() {
        return new MessageOriginProvider();
    }

}
```
{% endtab %}
{% endtabs %}

### Interceptor Logging <a id="interceptor-logging"></a>

Another good approach to track the flow of messages throughout an Axon application is by setting up the right interceptors in your application. There are two flavors of interceptors, the Dispatch and Handler Interceptors \(as discussed [here](messaging-concepts/message-intercepting.md)\), which intercept a message prior to publishing \(Dispatch Interceptor\) or while it is being handled \(Handler Interceptor\). The interceptor mechanism lends itself quite nicely to introduce a way to consistently log when a message is being dispatched/handled. The `LoggingInterceptor` is an out of the box solution to log any type of message to SLF4J, but also provides a simple overridable template to set up your own desired logging format. We refer to the command, event and query sections for the specifics on how to configure message interceptors.‌

### Event Tracker Status <a id="event-tracker-status"></a>

Since [Tracking Tokens](events/event-processors.md#token-store) "track" the progress of a given Tracking Event Processor, they provide a sensible monitoring hook in any Axon application. Such a hook proves its usefulness when we want to rebuild our view model and we want to check when the processor has caught up with all the events.‌

To that end the `TrackingEventProcessor` exposes the `processingStatus()` method. It returns a map where the key is the segment identifier and the value is an "Event Tracker Status". The Event Tracker Status exposes a couple of metrics:‌

* The `Segment` it reflects the status of.
* A boolean through `isCaughtUp()` specifying whether it is caught up with the Event Stream.
* A boolean through `isReplaying()` specifying whether the given Segment is ​[replaying](events/event-processors.md#replaying-events).

* A boolean through `isMerging()` specifying whether the given Segment is [​merging](events/event-processors.md#splitting-and-merging-tracking-tokens).
* The `TrackingToken` of the given Segment.
* A boolean through `isErrorState()` specifying whether the Segment is in an error state.
* An optional `Throwable` if the Event Tracker reached an error state.
* An optional `Long` through `getCurrentPosition` defining the current position of the `TrackingToken`.
* An optional `Long` through `getResetPosition` defining the position at reset of the `TrackingToken`.
  This field will be `null` in case the `isReplaying()` returns `false`.
  It is possible to derive an estimated duration of replaying by comparing the current position with this field.
* An optional `Long` through `mergeCompletedPosition()` defining the position on the `TrackingToken` when merging will be completed.
  This field will be `null` in case the `isMerging()` returns `false`.
  It is possible to derive an estimated duration of merging by comparing the current position with this field.

Some scenarios call for a means to react on _when_ the status' of a processor changes.
For example, whenever the status switches from replay being `true` or `false`.
To that end, a `EventTrackerStatusChangeListener` can be configured through the `TrackingEventProcessorConfiguration` for a `TrackingEventProcessor`.

The `EventTrackerStatusChangeListener` is a functional interface defining an `onEventTrackerStatusChange(Map<Integer, EventTrackerStatus>)` method, which will be invoked by the `TrackingEventProcessor` whenever there is a significant change in any one of the `EventTrackerStatus` objects.
The collection of integer to `EventTrackerStatus` provides the status' which have caused the change listener to be invoked.
This thus allows you to check the given `EventTrackerStatus`' to react accordingly.

Know that by default, the processor will only invoke the change listener if any of the boolean fields has changed.
If it is required to react on the position changes as well, you can provide a `EventTrackerStatusChangeListener` which overrides the `validatePositions` method to return `true`.
Do note that this means the change listener will be invoked _often_, as it is expected to handle lots of events.   

## Metrics <a id="metrics"></a>

Interesting metrics in a message centric system come in several forms and flavors, like count, capacity and latency for example. Axon Framework allows you to retrieve such measurements through the use of the `axon-metrics` or `axon-micrometer` module. With these modules you can register a number of `MessageMonitor` implementations to your messaging components, like the [`CommandBus`](axon-framework-commands/command-dispatchers.md#the-command-bus), [`EventBus`](events/event-bus-and-event-store.md#event-bus), [`QueryBus`](queries/query-dispatchers.md#the-query-bus-and-query-gateway) and [`EventProcessors`](events/event-processors.md).‌

`axon-metrics` module uses [Dropwizard Metrics](https://metrics.dropwizard.io/) for registering the measurements correctly. That means that `MessageMonitors` are registered against the Dropwizard `MetricRegistry`.‌

`axon-micrometer` module uses [Micrometer](https://micrometer.io/) which is a dimensional-first metrics collection facade whose aim is to allow you to time, count, and gauge your code with a vendor neutral API. That means that `MessageMonitors` are registered against the Micrometer `MeterRegistry`.‌

The following monitor implementations are currently provided:‌

1. `CapacityMonitor` - Measures message capacity by keeping track of the total time spent on message handling compared to total time it is active.

   This returns a number between 0 and n number of threads. Thus, if there are 4 threads working, the maximum capacity is 4 if every thread is active 100% of the time.

2. `EventProcessorLatencyMonitor` - Measures the difference in message timestamps between the last ingested and the last processed event message.
3. `MessageCountingMonitor` - Counts the number of ingested, successful, failed, ignored and processed messages.
4. `MessageTimerMonitor` - Keeps a timer for all successful, failed and ignored messages, as well as an overall timer for all three combined.
5. `PayloadTypeMessageMonitorWrapper` - A special `MessageMonitor` implementation which allows setting a monitor per message type instead of per message publishing/handling component.

‌

You are free to configure any combination of `MessageMonitors` through constructors on your messaging components, simply by using the Configuration API. The `GlobalMetricRegistry` contained in the `axon-metrics` and `axon-micrometer` modules provides a set of sensible defaults per type of messaging component. The following example shows you how to configure default metrics for your message handling components:‌

### Dropwizard

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class MetricsConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    // The MetricRegistry is a class from the Dropwizard Metrics framework
    public void configureDefaultMetrics(Configurer configurer, MetricRegistry metricRegistry) {
        GlobalMetricRegistry globalMetricRegistry = new GlobalMetricRegistry(metricRegistry);
        // We register the default monitors to our messaging components by doing the following
        globalMetricRegistry.registerWithConfigurer(configurer);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```text
# The default value is `true`. Thus you will have Metrics configured if `axon-metrics` and `io.dropwizard.metrics` are on your classpath.
axon.metrics.auto-configuration.enabled=true
```
{% endtab %}
{% endtabs %}

### Micrometer

{% tabs %}
{% tab title="Axon Configuration API - Without Tags" %}
```java
public class MetricsConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    // The MeterRegistry is a class from the Micrometer library
    public void configureDefaultMetrics(Configurer configurer, MeterRegistry meterRegistry) {
        GlobalMetricRegistry globalMetricRegistry = new GlobalMetricRegistry(meterRegistry);
        globalMetricRegistry.registerWithConfigurer(configurer);
    }
}
```
{% endtab %}

{% tab title="Axon Configuration API - With Tags" %}
```java
public class MetricsConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    // The MeterRegistry is a class from the Micrometer library
    public void configureDefaultMetrics(Configurer configurer, MeterRegistry meterRegistry) {
        GlobalMetricRegistry globalMetricRegistry = new GlobalMetricRegistry(meterRegistry);
        globalMetricRegistry.registerWithConfigurerWithDefaultTags(configurer);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Without Tags" %}
```text
# The default value is `true`. 
# Thus you will have Metrics configured if `axon-micrometer` and 
#  appropriate metric implementation (for example: `micrometer-registry-prometheus`) are on your classpath.
axon.metrics.auto-configuration.enabled=true

# Spring Boot metrics enabled
management.endpoint.metrics.enabled=true

# Spring Boot (Prometheus) endpoint (`/actuator/prometheus`) enabled and exposed
management.metrics.export.prometheus.enabled=true
management.endpoint.prometheus.enabled=true
```
{% endtab %}
{% tab title="Spring Boot AutoConfiguration - With Tags" %}
```text
# The default value is `true`. 
# Thus you will have Metrics configured if `axon-micrometer` and 
#  appropriate metric implementation (for example: `micrometer-registry-prometheus`) are on your classpath.
axon.metrics.auto-configuration.enabled=true

# The default value is `false`. 
# By enabling this property you will have message (event, command, query)
#   payload type set as a micrometer tag/dimension by default. 
# Additionally, the processor name will be a tag/dimension instead of it being part of the metric name.
axon.metrics.micrometer.dimensional=true

# Spring Boot metrics enabled
management.endpoint.metrics.enabled=true

# Spring Boot (Prometheus) endpoint (`/actuator/prometheus`) enabled and exposed
management.metrics.export.prometheus.enabled=true
management.endpoint.prometheus.enabled=true
```
{% endtab %}
{% endtabs %}

The scenario might occur that more fine-grained control over which `MessageMonitor` instance are defined is necessary.
The following snippet provides as sample if you want to have more specific metrics on any of the message handling components:

```java
// Java (Spring Boot Configuration) - Micrometer example
@Configuration
public class MetricsConfig {

    @Bean
    public ConfigurerModule metricConfigurer(MeterRegistry meterRegistry) {
        return configurer -> {
            instrumentEventStore(meterRegistry, configurer);
            instrumentEventProcessors(meterRegistry, configurer);
            instrumentCommandBus(meterRegistry, configurer);
            instrumentQueryBus(meterRegistry, configurer);
        };
    }

    private void instrumentEventStore(MeterRegistry meterRegistry, Configurer configurer) {
        MessageMonitorFactory messageMonitorFactory = (configuration, componentType, componentName) -> {
            MessageCountingMonitor messageCounter = MessageCountingMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName())
                                   .and(message.getMetaData().entrySet().stream()
                                               .map(s -> Tag.of(s.getKey(), s.getValue().toString()))
                                               .collect(Collectors.toList()))
            );
            // Naming the Timer monitor/meter with the name of the component (eventStore)
            // Registering the Timer with custom tags: payloadType.
            MessageTimerMonitor messageTimer = MessageTimerMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName())
            );
            return new MultiMessageMonitor<>(messageCounter, messageTimer);
        };
        configurer.configureMessageMonitor(EventStore.class, messageMonitorFactory);
    }

    private void instrumentEventProcessors(MeterRegistry meterRegistry, Configurer configurer) {
        MessageMonitorFactory messageMonitorFactory = (configuration, componentType, componentName) -> {

            // Naming the Counter monitor/meter with the fixed name `eventProcessor`.
            // Registering the Counter with custom tags: payloadType and processorName.
            MessageCountingMonitor messageCounter = MessageCountingMonitor.buildMonitor(
                    "eventProcessor", meterRegistry,
                    message -> Tags.of(
                            TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName(),
                            TagsUtil.PROCESSOR_NAME_TAG, componentName
                    )
            );
            // Naming the Timer monitor/meter with the fixed name `eventProcessor`.
            // Registering the Timer with custom tags: payloadType and processorName.
            MessageTimerMonitor messageTimer = MessageTimerMonitor.buildMonitor(
                    "eventProcessor", meterRegistry,
                    message -> Tags.of(
                            TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName(),
                            TagsUtil.PROCESSOR_NAME_TAG, componentName
                    )
            );
            // Naming the Capacity/Gauge monitor/meter with the fixed name `eventProcessor`.
            // Registering the Capacity/Gauge with custom tags: payloadType and processorName.
            CapacityMonitor capacityMonitor1Minute = CapacityMonitor.buildMonitor(
                    "eventProcessor", meterRegistry,
                    message -> Tags.of(
                            TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName(),
                            TagsUtil.PROCESSOR_NAME_TAG, componentName
                    )
            );

            return new MultiMessageMonitor<>(messageCounter, messageTimer, capacityMonitor1Minute);
        };
        configurer.configureMessageMonitor(TrackingEventProcessor.class, messageMonitorFactory);
    }

    private void instrumentCommandBus(MeterRegistry meterRegistry, Configurer configurer) {
        MessageMonitorFactory messageMonitorFactory = (configuration, componentType, componentName) -> {
            MessageCountingMonitor messageCounter = MessageCountingMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(
                            TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName(),
                            "messageId", message.getIdentifier()
                    )
            );
            MessageTimerMonitor messageTimer = MessageTimerMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName())
            );

            CapacityMonitor capacityMonitor1Minute = CapacityMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName())
            );

            return new MultiMessageMonitor<>(messageCounter, messageTimer, capacityMonitor1Minute);
        };
        configurer.configureMessageMonitor(CommandBus.class, messageMonitorFactory);
    }

    private void instrumentQueryBus(MeterRegistry meterRegistry, Configurer configurer) {
        MessageMonitorFactory messageMonitorFactory = (configuration, componentType, componentName) -> {
            MessageCountingMonitor messageCounter = MessageCountingMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(
                            TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName(),
                            "messageId", message.getIdentifier()
                    )
            );
            MessageTimerMonitor messageTimer = MessageTimerMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName())
            );
            CapacityMonitor capacityMonitor1Minute = CapacityMonitor.buildMonitor(
                    componentName, meterRegistry,
                    message -> Tags.of(TagsUtil.PAYLOAD_TYPE_TAG, message.getPayloadType().getSimpleName())
            );

            return new MultiMessageMonitor<>(messageCounter, messageTimer, capacityMonitor1Minute);
        };
        configurer.configureMessageMonitor(QueryBus.class, messageMonitorFactory);
    }
}
```

