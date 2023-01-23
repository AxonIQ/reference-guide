# Heartbeat Monitoring

You can enable the heartbeat monitoring in Axon Server in order to activate an high level monitoring of client connection availability.

As gRPC already provides an internal heartbeat, this feature is to cover the scenarios for which the gRPC implementation does not suffice.

This feature can be enabled by configuring the following property:

```text
axoniq.axonserver.heartbeat.enabled=true
```

Please note that when combining Axon Server with Axon Framework, the framework application should also have Heartbeat Monitoring enabled.
Note that this is enabled *by default* in Axon Framework.
If you want to emphasize this further, regard the following configuration:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void enableHeartbeats(Configurer configurer) {
        configurer.registerComponent(AxonServerConfiguration.class, config -> {
            AxonServerConfiguration.HeartbeatConfiguration heartbeatConfig =
                    new AxonServerConfiguration.HeartbeatConfiguration();
            heartbeatConfig.setEnabled(true);

            AxonServerConfiguration serverConfig = new AxonServerConfiguration();
            serverConfig.setHeartbeat(heartbeatConfig);
            return serverConfig;
        });
    }
}
```
{% endtab %}

{% tab title="Spring Boot Auto Configuration" %}
```text
axon.axonserver.heartbeat.enabled=true
```
{% endtab %}
{% endtabs %}

> **Axon Framework Compatibility**
>
> Axon Framework supports heartbeat monitoring starting from 4.2.1 version.
