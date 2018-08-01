# Metrics & Monitoring

Important in any application is the ability to monitor and measure what is going on. 
Especially in a location transparent environment like an Axon application it is very important to be able to trace your message and check the ingestion rate of it.

## Metrics

Interesting metrics in a message centric system come in several forms and flavors, like count, capacity and latency for example.  
The Axon Framework allows you to retrieve such measurements through the use of the `axon-metrics` module.
With this module you can registers a number of `MessageMonitor` implementations to your messaging components, like the [`CommandBus`](../part-iii-infrastructure-components/command-dispatching.md#the-command-bus), [`EventBus`](../part-iii-infrastructure-components/event-processing.md#event-bus), [`QueryBus`](../part-iii-infrastructure-components/query-processing.md#query-bus) and [`EventProcessors`](../part-iii-infrastructure-components/event-processing.md#event-processors).

Internally, the `axon-metrics` module uses the [Dropwizard Metrics](https://metrics.dropwizard.io/) for registering the measurements correctly.
That thus means that `MessageMonitors` are registered against the Dropwizard `MetricRegistry`.
The following monitor implementations are currently provided:
1. `CapacityMonitor` - Calculates capacity by tracking, within a configurable time window, the average message processing time and multiplying that by the amount of messages processed.
2. `EventProcessorLatencyMonitor` - Measures the difference in message timestamps between the last ingested and the last processed event message.
3. `MessageCountingMonitor` - Counts the number of ingested, successful, failed, ignored and processed messages.
4. `MessageTimerMonitor` - Keeps a timer for all successful, failed and ignored messages, as well as an overall timer for all three combined.
5. `PayloadTypeMessageMonitorWrapper` - A special `MessageMonitor` implementation which allows setting a monitor per message type instead of per message publishing/handling component. 

You are free to configure any combination of `MessageMonitors` through constructors on your messaging components, and even simpler by using the [Configuration API](../part-i-getting-started/configuration-api.md).
The `GlobalMetricRegistry` contained in the `axon-metrics` module provides a set of sensible defaults per type of messaging component.
The following example shows you how to configure default metrics for your components, but also a specific metrics set for your EventBus: 

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

    public void configureSpecificEventBusMetrics(Configurer configurer, MetricRegistry metricRegistry) {
        // So all our messaging components are now set, but we want a different set of metrics for our EventBus
        // More specifically, we want to count the messages per type of event being published.
        PayloadTypeMessageMonitorWrapper<MessageCountingMonitor> messageCounterPerType =
                new PayloadTypeMessageMonitorWrapper<>(MessageCountingMonitor::new);
        // Add we also want to set a message timer per payload type
        PayloadTypeMessageMonitorWrapper<MessageTimerMonitor> messageTimerPerType =
                new PayloadTypeMessageMonitorWrapper<>(MessageTimerMonitor::new);
        // Which we group in a MultiMessageMonitor
        MultiMessageMonitor<Message<?>> multiMessageMonitor =
                new MultiMessageMonitor<>(messageCounterPerType, messageTimerPerType);
        // And configure through the Configuration API to every EventBus component
        configurer.configureMessageMonitor(EventBus.class, configuration -> multiMessageMonitor);

        // But do not forget to register them to the global MetricRegistry
        MetricRegistry eventBusRegistry = new MetricRegistry();
        eventBusRegistry.register("messageCounterPerType", messageCounterPerType);
        eventBusRegistry.register("messageTimerPerType", messageTimerPerType);
        metricRegistry.register("eventBus", eventBusRegistry);
    }
    
} 
```


## Monitoring



  