# Actuator Endpoints

For monitoring, AxonServer includes Spring Boot Actuator endpoints which are available under the `/actuator` URL path of the AxonServer.‌ For Axon Server SE, the URL for the Axon Server SE will be the single running node, while for Axon Server EE the URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.

The `/actuator`URL path returns a list of all available actuators. A list of the endpoints is given below.

* `/actuator/health` endpoint is used to check the health of Axon Server itself, and \(in the case of Axon Server EE\) the health of the cluster. The HTTP status return code _**is not 200**_ when cluster health is anything other than "UP".
* `/actuator/info` endpoint informs you about some basic attributes of an AxonServer \(name, description, version\). This is more useful for liveness/readiness probes.
* `/actuator/metrics` endpoint publishes information about OS, JVM as well as [application level metrics​](./#application-level-metrics)
* `/actuator/loggers` endpoint exposes detailed view of the loggers configuration
* `/actuator/prometheus` endpoint exposes metrics data in a format that can be scraped by a [Prometheus server \(Monitoring system & time series database\)](https://prometheus.io/)​
* `/actuator/env` endpoint exposes the current environment properties

## Application level metrics‌ <a id="application-level-metrics"></a>

Specific AxonServer metrics is available under `/actuator/metrics` endpoint:‌

* `/actuator/metrics/axon.events.count` current number of events stored in a node
* `/actuator/metrics/axon.commands.count` current number of commands stored in a node
* `/actuator/metrics/axon.queries.count` current number of queries stored in a node
* `/actuator/metrics/axon.snapshots.count` current number of snapshots stored in a node
* `/actuator/metrics/axon.commands.active` current number of active commands in a node

Other, AxonServer-specific endpoints can be browsed through `/swagger-ui.html`.

