# Heartbeat Monitoring

You can enable the heartbeat monitoring in Axon Server in order to activate an high level monitoring of client connection availability.

Axon Framework and Axon Server need to know whether the connection is still alive.
GRPC already provides an internal heartbeat in its protocol but does not suffice in many situations, such as in the cloud or when using a service mesh. Proxies and Load Balancers understand this heartbeat and respond to it, while the connection beyond that's connected to Axon Server is not checked properly. 

For this reason, the API of Axon Server implements a  non-protocol heartbeat that is a regular gRPC call, sent at intervals. If the call is not responded to in time, the connection is terminated and re-established, providing recoverability from various network issues. Therefore it's enabled by default. 



This feature can be enabled by configuring the following property:

```text
axoniq.axonserver.heartbeat.enabled=true
```

Please note that when combining Axon Server with Axon Framework, the framework application should also have Heartbeat Monitoring enabled.
Note that this is enabled *by default* in Axon Framework.

If you want to disable heartbeat monitoring this further, regard the following configuration:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class AxonConfig {
    // ...
    public void disableHeartbeats(Configurer configurer) {
        configurer.registerComponent(AxonServerConfiguration.class, config -> {
            AxonServerConfiguration.HeartbeatConfiguration heartbeatConfig =
                    new AxonServerConfiguration.HeartbeatConfiguration();
            heartbeatConfig.setEnabled(false);

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
axon.axonserver.heartbeat.enabled=false
```
{% endtab %}
{% endtabs %}

> **Axon Framework Compatibility**
>
> Axon Framework supports heartbeat monitoring starting from 4.2.1 version.
