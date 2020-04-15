# Heartbeat Monitoring

You can enable the heartbeat monitoring in Axon Server in order to activate an high level monitoring of client connection availability.‌

As gRPC already provides an internal heartbeat, this feature is to cover the scenarios for which the gRPC implementation does not suffice.‌

This feature can be enabled by configuring the following property:

```text
axoniq.axonserver.heartbeat.enabled=true
```

‌

Please note that, in order to have this feature properly working, also the client should enable the Heartbeat Monitoring functionality.‌

The feature is disable by default; it can be enabled in the following way:Spring Boot AutoConfiguration

```text
Configurer configurer = DefaultConfigurer.defaultConfiguration()                                         .registerModule(new HeartbeatConfiguration());
```

> **Axon Framework Compatibility**
>
> Axon Framework supports heartbeat monitoring starting from 4.2.1 version.
