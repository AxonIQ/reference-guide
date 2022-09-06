# Health indicators

Axon Framework provides health indicators for applications running Spring boot with the actuator module. 
These can be used to monitor the health of your application.

### Spring Boot Actuator Health Indicator for Axon Server

The Spring Boot Actuator HealthIndicator shares whether the contexts to which an Axon Framework application is connected with are active.
It does so by requesting the available connections from the `AxonServerConnectionManager`.

When all connections are active, the UP status is shared.
When all connections are inactive, the DOWN status is projected.
When one of the connections is inactive, the custom WARN status is shown.
This approach is in line with what Axon Server's local health indicator shows.

Next to the status, details are provided about the separate connection's activity.
These can be found under `{context-name}`.connection.active.
