# Serializers

Event stores need a way to serialize the event to prepare it for storage. 
By default, Axon uses the `XStreamSerializer`, which uses [XStream](http://x-stream.github.io/) to serialize events into XML. 
XStream is reasonably fast and is more flexible than Java Serialization. 
Furthermore, the result of XStream serialization is human readable. Quite useful for logging and debugging purposes.

The `XStreamSerializer` can be configured. 
You can define aliases it should use for certain packages, classes or even fields. 
Besides being a nice way to shorten potentially long names, aliases can also be used when class definitions of events change. 
For more information about aliases, visit the [XStream website](http://x-stream.github.io/).

Alternatively, Axon also provides the `JacksonSerializer`,
 which uses [Jackson](https://github.com/FasterXML/jackson) to serialize events into JSON. 
While it produces a more compact serialized form,
 it does require that classes stick to the conventions \(or configuration\) required by Jackson.

You may also implement your own serializer, simply by creating a class that implements `Serializer`,
 and configuring the event store to use that implementation instead of the default.

## Serializer Implementations

Serializers come in several flavors in the Axon Framework and are used for a variety of subjects. 
Currently you can choose between the `XStreamSerializer`,
 `JacksonSerializer` and `JavaSerializer` to serialize the messages \(commands/queries/events\), tokens,
  snapshots and sagas in an Axon application.

As there are several objects to be serialized, it is typically desired to tune which serializer handles which. 
To that end, the `Configuration` API allows you to define a default, message and event serializer,
 which lead to the following object-serialization break down:

1. The Event `Serializer` is in charge of de-/serializing event messages. 
Events are typically stored in an event store for a long period of time. 
This is the main driver for choosing the event serializer implementation.
2. The Message `Serializer` is in charge of de-/serializing the command and query messages
 \(used in a distributed application set up\). 
Messages are shared between nodes and typically need to be interoperable and/or compact. 
Take this into account when choosing the message serializer. 
3. The default `Serializer` is in charge of de-/serializing the remainder, being the tokens, snapshots and sagas. 
These objects are generally not shared between different applications,
 and most of these classes aren't expected to have some of the getters and setters that are, for example,
 typically required by Jackson based serializers. 
A flexible, general purpose serializer like [XStream](http://x-stream.github.io/) is quite suited for this purpose.

By default all three `Serializer` flavors are set to use the `XStreamSerializer`,
 which internally uses [XStream](http://x-stream.github.io/) to serialize objects to an XML format. 
XML is a verbose format to serialize to, but XStream has the major benefit of being able to serialize virtually anything. 
This verbosity is typically fine when storing tokens, sagas or snapshots,
 but for messages \(and specifically events\) XML might prove to cost too much due to its serialized size. 
Thus for optimization reasons you can configure different serializers for your messages. 
Another very valid reason for customizing Serializers is to achieve interoperability between different \(Axon\) applications,
 where the receiving end potentially enforces a specific serialized format.

There is an implicit ordering between the configurable serializer. 
If no Event `Serializer` is configured, the Event de-/serialization will be performed by the Message `Serializer`. 
In turn, if no Message `Serializer` is configured, the default `Serializer` will take that role.

See the following example on how to configure each serializer specifically,
 were we use `XStreamSerializer` as the default and `JacksonSerializer` for all our messages:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class SerializerConfiguration {

    public void serializerConfiguration(Configurer configurer) {
        // Per default we want the XStreamSerializer
        XStreamSerializer defaultSerializer = new XStreamSerializer();
        // But for all our messages we'd prefer the JacksonSerializer due to JSON its smaller format
        JacksonSerializer messageSerializer = new JacksonSerializer();

        configurer.configureSerializer(configuration -> defaultSerializer)
                  .configureMessageSerializer(configuration -> messageSerializer)
                  .configureEventSerializer(configuration -> messageSerializer);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - Properties file" %}
```text
# Possible values for these keys are `default`, `xstream`, `java`, and `jackson`.
axon.serializer.general
axon.serializer.events
axon.serializer.messages
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration - YML file" %}
```yaml
# Possible values for these keys are `default`, `xstream`, `java`, and `jackson`.
axon:
    serializer:
        general: 
        events: 
        messages:
```
{% endtab %}
{% endtabs %}

## Serializing events vs 'the rest'

It is possible to use a different serializer for the storage of events,
 than all other objects that Axon needs to serializer \(such as commands, snapshots, sagas, etc\). 
While the `XStreamSerializer`'s capability to serialize virtually anything makes it a very decent default,
 its output is not always a form that makes it nice to share with other applications. 
The `JacksonSerializer` creates much nicer output, but requires a certain structure in the objects to serialize. 
This structure is typically present in events, making it a very suitable event serializer.

Using the Configuration API, you can simply register an event serializer as follows:

```java
Configurer configurer = ... // initialize
configurer.configureEventSerializer(conf -> /* create serializer here*/);
```

If no explicit `eventSerializer` is configured,
 Events are serialized using the main serializer that has been configured \(which in turn defaults to the `XStreamSerializer`\).