# Monitoring

For monitoring, AxonServer includes Spring Boot Actuator endpoints which are available under the `/actuator` URL path of the AxonServer.

In particular, you will find following actuator endpoints enabled:

* `/actuator/health` endpoint is used to check the health or state of the AxonServer
* `/actuator/info` endpoint informs you about some basic attributes of an AxonServer \(name, description, version\)
* `/actuator/metrics` endpoint publishes information about OS, JVM as well as [application level metrics](monitoring.md#application-level-metrics)
* `/actuator/loggers` endpoint exposes detailed view of the loggers configuration
* `/actuator/prometheus` endpoint exposes metrics data in a format that can be scraped by a [Prometheus server \(Monitoring system & time series database\)](https://prometheus.io/)
* `/actuator/env` endpoint exposes the current environment properties 

## Application level metrics

Specific AxonServer metrics is available under `/actuator/metrics` endpoint:

* `/actuator/metrics/axon.events.count` current number of events stored in a node
* `/actuator/metrics/axon.commands.count` current number of commands stored in a node
* `/actuator/metrics/axon.queries.count` current number of queries stored in a node
* `/actuator/metrics/axon.snapshots.count` current number of snapshots stored in a node
* `/actuator/metrics/axon.commands.active` current number of active commands in a node

Other, AxonServer-specific endpoints can be browsed through `/swagger-ui.html`.

