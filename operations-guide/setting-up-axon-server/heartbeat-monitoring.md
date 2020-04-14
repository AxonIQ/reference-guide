# Heartbeat Monitoring

In certain situations, it is not desirable to exclusively use transport level heartbeats. The Axon Connector and Server supper application-level heartbeat monitoring, which validates end-to-end connectivity at the cost of some I/O overhead.

To use application-level heartbeats, the feature must be enabled on both AxonServer as the connected clients.

In AxonServer, heartbeats are configured by adding the following property in `axonserver.properties`:

```text
axoniq.axonserver.heartbeat.enabled=true
```

Please note that, in order to have this feature properly working, also the client should enable the Heartbeat Monitoring functionality.

The feature is disable by default; it can be enabled using the Configuration API:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
                                         .registerModule(new HeartbeatConfiguration());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
by configuring the following property:

```text
axon.axonserver.heartbeat.auto-configuration.enabled=true
```
{% endtab %}
{% endtabs %}

> **Axon Framework Compatibility**
>
> Axon Framework supports heartbeat monitoring starting from 4.2.1 version.

