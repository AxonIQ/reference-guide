# Monitoring and Metrics

The ability to monitor and measure what is going on is very important. 
Especially in a location transparent environment like an Axon application
 it is very important to be able to trace your message and check the ingestion rate of it.

## Monitoring

Monitoring a message centric application will require you to be able to see where your messages are at a given point in time. 
This translates to being able to track your commands,
 events and queries from one component to another in an Axon application.

### Correlation Data

One import aspect in regards to this is tracing a given message. 
To that end the framework provides the `CorrelationDataProvider`,
 as described briefly [here](../../configuring-infrastructure-components/messaging-concepts/message-intercepting.md). 
This interface and its implementations provide you the means to populate the meta-data of your messages with specific fields,
 like a 'trace-id', 'correlation-id' or any other field you might be interested in.

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
```java
public class MonitoringConfiguration {

    // When using Spring Boot, simply defining a CorrelationDataProvider bean is sufficient
    public CorrelationDataProvider messageOriginProvider() {
        return new MessageOriginProvider();
    }

}
```
{% endtab %}
{% endtabs %}

### Interceptor Logging

Another good approach to track the flow of messages throughout an Axon application is by setting up the right interceptors in your application.  
There are two flavors of interceptors, the Dispatch and Handler Interceptors
 \(like discussed [here](../../configuring-infrastructure-components/messaging-concepts/message-intercepting.md)\), 
 which intercept a message prior to publishing \(Dispatch Interceptor\) or whilst it is being handled \(Handler Interceptor\). 
The interceptor mechanism lends itself quite nicely to introduce a way to consistently log when a message is being dispatched/handled. 
The `LoggingInterceptor` is an out of the box solution to log any type of message to SLF4J,
 but also provides a simple overridable template to set up your own desired logging format. 
We refer to the command, event and query sections for the specifics on how to configure message interceptors.

## Metrics

Interesting metrics in a message centric system come in several forms and flavors, like count, capacity and latency for example.  
The Axon Framework allows you to retrieve such measurements through the use of the `axon-metrics` module.
With this module you can register a number of `MessageMonitor` implementations to your messaging components,
 like the [`CommandBus`](../../configuring-infrastructure-components/command-processing/command-dispatching.md#the-command-bus),
 [`EventBus`](../../configuring-infrastructure-components/event-processing/event-bus-and-event-store.md#event-bus),
 [`QueryBus`](../../configuring-infrastructure-components/query-processing.md#query-bus)
 and [`EventProcessors`](../../configuring-infrastructure-components/event-processing/event-processors.md#event-processors).

Internally, the `axon-metrics` module uses [Dropwizard Metrics](https://metrics.dropwizard.io/) for registering the measurements correctly. 
That thus means that `MessageMonitors` are registered against the Dropwizard `MetricRegistry`. 
The following monitor implementations are currently provided:

1. `CapacityMonitor` - Measure the message capacity by keeping track of the total time spent on message handling compared to total time it is active. 
This returns a number between 0 and n number of threads, thus if there are 4 threads working, the maximum capacity is 4 if every thread is active for 100% 
2. `EventProcessorLatencyMonitor` - Measures the difference in message timestamps between the last ingested and the last processed event message.
3. `MessageCountingMonitor` - Counts the number of ingested, successful, failed, ignored and processed messages.
4. `MessageTimerMonitor` - Keeps a timer for all successful, failed and ignored messages, as well as an overall timer for all three combined.
5. `PayloadTypeMessageMonitorWrapper` - A special `MessageMonitor` implementation which allows setting a monitor per message type instead of per message publishing/handling component. 

You are free to configure any combination of `MessageMonitors` through constructors on your messaging components,
 and even simpler by using the Configuration API. 
The `GlobalMetricRegistry` contained in the `axon-metrics` module provides a set of sensible defaults per type of messaging component. 
The following example shows you how to configure default metrics for your message handling components:

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
# The default value is `true`. 
# Thus you will have Metrics configured if axon-metrics and io.dropwizard.metrics are on your classpath.
axon.metrics.auto-configuration.enabled=true
```
{% endtab %}
{% endtabs %}

If you want to have more specific metrics on a message handling component like the `EventBus`, you can do the following:

```java
public class MetricsConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    public void configureSpecificEventBusMetrics(Configurer configurer, MetricRegistry metricRegistry) { 
        // For the EventBus we want to count the messages per type of event being published.
        PayloadTypeMessageMonitorWrapper<MessageCountingMonitor> messageCounterPerType =
                new PayloadTypeMessageMonitorWrapper<>(MessageCountingMonitor::new);
        // And we also want to set a message timer per payload type
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
