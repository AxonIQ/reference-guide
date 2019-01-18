# Monitoring

Monitoring a message centric application will require you to be able to see where your messages are at a given point in time. 
This translates to being able to track your commands, events and queries from one component to another in an Axon application.

## Correlation Data

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

## Interceptor Logging

Another good approach to track the flow of messages throughout an Axon application is by setting up the right interceptors in your application.  
There are two flavors of interceptors, the Dispatch and Handler Interceptors
 \(like discussed [here](../../configuring-infrastructure-components/messaging-concepts/message-intercepting.md)\), 
 which intercept a message prior to publishing \(Dispatch Interceptor\) or whilst it is being handled \(Handler Interceptor\). 
The interceptor mechanism lends itself quite nicely to introduce a way to consistently log when a message is being dispatched/handled. The `LoggingInterceptor` is an out of the box solution to log any type of message to SLF4J, but also provides a simple overridable template to set up your own desired logging format. We refer to the command, event and query sections for the specifics on how to configure message interceptors.