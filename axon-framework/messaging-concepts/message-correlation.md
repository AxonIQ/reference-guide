# Message correlation

In messaging systems it is common to group messages together, or correlate. In Axon Framework a `Command` message might result in one or several
`Event` messages and a `Query` message might result in one or several `QueryResult` messages. Usually, correlation is implemented using a specific
message property, a so called correlation identifier.

## Correlation Data Provider

Messages in Axon Framework use `MetaData` property to transport meta information about the message. `MetaData` is of type `Map<String, Object>` and is passed around
with the message. In order to fill the `MetaData` of a message produced in a Unit of Work, a correlation data provider can be used.

### MessageOriginProvider

By default, the `MessageOriginProvider` is registered as a correlation data provider. It is responsible for two values `correlationId` and `traceId` being passed
around from `Command` message to `Event` message and from `Query` message to `QueryResult` message. The default for the `correlationId` and `traceId` is the message identifier.

### SimpleCorrelationDataProvider

The `SimpleCorrelationDataProvider` is configured to unconditionally copy values of specified keys from one message to metadata of the other. To do so, the constructor of the
`SimpleCorrelationDataProvider` must be called with a list of keys which should be copied. Here is an example, how to configure it to copy the `myId` and `myId2` values.

```java
SimpleCorrelationDataProvider provider = new SimpleCorrelationDataProvider("myId", "myId2");
```

### MultiCorrelationDataProvider

A `MultiCorrelationDataProvider` is capable to combine the effect of multiple correlation data providers. To do so, the constructor of the `MultiCorrelationDataProvider` must be
called with a list of providers. Here is an example:

```java

MultiCorrelationDataProvider<CommandMessage<?>> provider = new MultiCorrelationDataProvider<CommandMessage<?>>(
    Arrays.asList(
        new SimpleCorrelationDataProvider("someKey"),
        new MessageOriginProvider()
    )
);
```

### Implementing custom correlation data provider

If the predefined providers don't fulfill your requirements, you can implement your own correlation data provider. The class must implement the `CorrelationDataProvider` interface. Here is a simple example:

```java

public class AuthCorrelationDataProvider implements CorrelationDataProvider {

  private final Function<String, String> usernameProvider;

  public AuthCorrelationDataProvider(Function<String, String> userProvider) {
    this.usernameProvider = userProvider;
  }

  @Override
  public Map<String, ?> correlationDataFor(Message<?> message) {
    Map<String, Object> map = new HashMap<>();
    if (message instanceof CommandMessage<?>) {
      if (message.getMetaData().containsKey("authorization")) {
        String token = (String) message.getMetaData().get("authorization");
        map.put("username", usernameProvider.apply(token));
      }
    }
    return map;
  }
}
```

## Configuration

The correlation data providers need to be registered inside you application.
If you are using Axon Configuration API, make sure to call `Configuration#configureCorrelationDataProviders`
method to register the correlated data providers. If you are relying on Spring Boot
Autoconfiguration, just provide a factory method exposing the Spring Bean or multiple,
if you need more than one provider. The following snippets shows some possible approaches of registration:


{% tabs %}

{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
       .configureCorrelationDataProviders(c -> Arrays.asList(
         new SimpleCorrelationDataProvider("someKey"),
         new MessageOriginProvider()
       ));
}
```
{% endtab %}

{% tab title="Spring Boot Autoconfiguration" %}
```java

@Configuration
public class CorrelationDataProviderConfiguration {

  @Bean
  public CorrelationDataProvider myCorrelationDataProvider() {
    return new MultiCorrelationDataProvider<CommandMessage<?>>(
        Arrays.asList(
          new SimpleCorrelationDataProvider("someKey"),
          new MessageOriginProvider()
        )
    );
  }
}

```
{% endtab %}

{% endtabs %}
