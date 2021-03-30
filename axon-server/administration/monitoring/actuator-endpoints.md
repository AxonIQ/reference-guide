# Actuator Endpoints

For monitoring, AxonServer includes Spring Boot Actuator endpoints which are available under the `/actuator` URL path of the AxonServer.‌ For Axon Server SE, the URL for the Axon Server SE will be the single running node, while for Axon Server EE the URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.

The `/actuator`URL path returns a list of all available actuators. A list of the endpoints is given below.

* `/actuator/health` endpoint is used to check the health of Axon Server itself, and \(in the case of Axon Server EE\) the health of the cluster. The HTTP status return code _**is not 200**_ when cluster health is anything other than "UP".
* `/actuator/info` endpoint informs you about some basic attributes of an AxonServer \(name, description, version\). This is more useful for liveness/readiness probes.
* `/actuator/metrics` endpoint publishes information about OS, JVM as well as [application level metrics​](./#application-level-metrics)
* `/actuator/loggers` endpoint exposes detailed view of the loggers configuration
* `/actuator/prometheus` endpoint exposes metrics data in a format that can be scraped by a [Prometheus server \(Monitoring system & time series database\)](https://prometheus.io/)​
* `/actuator/env` endpoint exposes the current environment properties

### Health endpoint

The /actuator/health endpoint provides information on various components of Axon Server:
* cluster (EE only), shows the status of the connection between the current Axon Server node and other nodes in the cluster. This section also shows information on the flow control
  between Axon Server nodes. If the value for _commands.waiting_ or _queries.waiting_ is not zero for a longer period of time, it means that the connected Axon Server node cannot 
  process the commands or queries fast enough. If there are waiting commands or queries, or the connection with one of the other nodes is down, this component will show status _WARN_. 
  If the connections to all other nodes is down, the status of the cluster component is _DOWN_.
* command, shows the status of command handling applications connected to the current Axon Server node. For each connected application it shows the number of commands waiting to 
  be sent to the command handler. If this value is higher than zero, it means that commands are being sent faster than the command handler can handle them.
* db, shows the status of the control DB
* diskSpace, shows the diskspace in use and available for each storage location (this includes the locations where events and snapshots are stored, and the location of the replication logs).
  if the available space in any of these locations is below the threshold, the health for the diskSpace component will be _WARN_.
* license (EE only, since 4.5). shows the status of the license. If the license is in the grace period, this component gets status _WARN_, if license is expired the status becomes _DOWM_.
* localEventStore, shows the status of the local replica for each context available on the current node. 
* ping, always shows UP
* query, shows the status of query handling applications connected to the current Axon Server node. For each connected application it shows the number of queries waiting to
  be sent to the query handler. If this value is higher than zero, it means that queries are being sent faster than the query handler can handle them.
* raft, shows the status of the replication process per replication group. It shows the leader of the replication group. For the leader it shows the status of the followers, checking if the follower has recently confirmed messages and if the follower is not too far behind.
  If there is a problem with the replication on one of the replication groups the health for this component is _WARN_. If there are issues for all replication groups the status is _DOWN_. 

The overall health status is derived from the status of the components. If one of the components is _WARN_, the overall status is _WARN_. If one of the components is _DOWN_, it is _DOWN_.

The default setting for the health endpoint in Axon Server is to show the details, even when the user is not authenticated. To hide the details for non-authenticated users add this property 
to the axonserver.properties file:
```bash
management.endpoint.health.show-details=when-authorized
```

## Application level metrics‌ <a id="application-level-metrics"></a>

Specific AxonServer metrics are available under `/actuator/metrics` endpoint:‌

* `/actuator/metrics/axon.events.count` current number of events stored in a node
* `/actuator/metrics/axon.commands.count` current number of commands stored in a node
* `/actuator/metrics/axon.queries.count` current number of queries stored in a node
* `/actuator/metrics/axon.snapshots.count` current number of snapshots stored in a node
* `/actuator/metrics/axon.commands.active` current number of active commands in a node



