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

## Application level metrics <a id="application-level-metrics"></a>

Axon Server provides metrics to inspect the state of the process. A full list of all the active metrics is available under the `/actuator/metrics` endpoint. The next sections give an overview of the metrics related to client request handling.  

### Command handling metrics

| Metric name                                     | Description                                                                         |
|-------------------------------------------------|-------------------------------------------------------------------------------------| 
| axon.commands.throughput.count                  | number of commands processed since the start of the server                          |
| axon.commands.throughput.rate.oneMinuteRate     | average number of commands per second processed in the last minute                  |
| axon.commands.throughput.rate.fiveMinuteRate    | average number of commands per second processed in the five minutes                 |
| axon.commands.throughput.rate.fifteenMinuteRate | average number of commands per second processed in the fifteen minutes              |
| axon.commands.duration                          | Duration of commands, from received by Axon Server until response sent to requester |
| axon.commands.duration.handling                 | Duration of commands, from sent to handler until response received from handler     |
| axon.commands.error.count                       | Number of errors                                                                    | 
| axon.commands.saturation.queued.count           | Number of commands queued in Axon Server                                            |
| axon.commands.saturation.active.count           | Number of in-flight commands                                                        |

### Query handling metrics

| Metric name                                       | Description                                                                        |
|---------------------------------------------------|------------------------------------------------------------------------------------|
| axon.queries.throughput.count                     | number of queries processed since the start of the server                          |
| axon.queries.throughput.rate.oneMinuteRate        | average number of queries per second processed in the last minute                  |
| axon.queries.throughput.rate.fiveMinuteRate       | average number of queries per second processed in the five minutes                 |
| axon.queries.throughput.rate.fifteenMinuteRate    | average number of queries per second processed in the fifteen minutes              |
| axon.queries.duration                             | Duration of queries, from received by Axon Server until response sent to requester |
| axon.queries.duration.handling                    | Duration of queries, from sent to handler until response received from handler     |
| axon.queries.error.count                          | Number of errors                                                                   |
| axon.queries.saturation.queued.count              | Number of queries queued in Axon Server                                            |
| axon.queries.saturation.active.count              | Number of in-flight queries                                                        |
| axon.queries.subscriptionquery.throughput.total   | Total number of subscription queries subscribed                                    |
| axon.queries.subscriptionquery.duration           | Duration of a subscription query connection                                        |
| axon.queries.subscriptionquery.throughput.updates | Total number of updates submitted on subscription queries                          |
| axon.queries.subscriptionquery.saturation.active  | Active number of subscription queries on this node                                 |


### Event store metrics

| Metric name                                                  | Description                                                                                                                         |
|--------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| axon.events.append.throughput.count                          | number of events appended since the start of the server                                                                             |
| axon.events.append.throughput.rate.oneMinuteRate             | average number of events appended per second processed in the last minute                                                           |
| axon.events.append.throughput.rate.fiveMinuteRate            | average number of events appended per second processed in the five minutes                                                          |
| axon.events.append.throughput.rate.fifteenMinuteRate         | average number of events appended  per second processed in the fifteen minutes                                                      |
| axon.events.append.duration                                  | Duration of append events request, from the first event in a transaction received by Axon Server until the transaction is completed |
| axon.events.append.error.count                               | Number of errors appending events                                                                                                   | 
| axon.events.append.saturation.active.count                   | Number of active append event transactions                                                                                          |
| axon.events.read.aggregate.throughput.count                  | number of aggregates read d since the start of the server                                                                           |
| axon.events.read.aggregate.throughput.rate.oneMinuteRate     | average number of aggregates read per second processed in the last minute                                                           |
| axon.events.read.aggregate.throughput.rate.fiveMinuteRate    | average number of aggregates read per second processed in the five minutes                                                          |
| axon.events.read.aggregate.throughput.rate.fifteenMinuteRate | average number of aggregates read per second processed in the fifteen minutes                                                       |
| axon.events.read.aggregate.duration                          | Duration of read aggregate request                                                                                                  |
| axon.events.read.aggregate.error.count                       | Number of errors                                                                                                                    | 
| axon.events.read.aggregate.saturation.active.count           | Number of active aggregate read actions                                                                                             |
| axon.snapshots.append.throughput.count                       | number of snapshots appended since the start of the server                                                                          |
| axon.snapshots.append.throughput.rate.oneMinuteRate          | average number of snapshot appends per second processed in the last minute                                                          |
| axon.snapshots.append.throughput.rate.fiveMinuteRate         | average number of snapshot appends per second processed in the five minutes                                                         |
| axon.snapshots.append.throughput.rate.fifteenMinuteRate      | average number of snapshot appends per second processed in the fifteen                                                              |
| axon.snapshots.append.duration                               | Duration of append snapshot request, from the snapshot received by Axon Server until the transaction is completed                   |
| axon.snapshots.append.error.count                            | Number of errors                                                                                                                    |
| axon.snapshots.append.saturation.active.count                | Number of active append snapshot requests                                                                                           |
| axon.snapshots.read.throughput.count                         | Number of snapshots read since the start of the server                                                                              |
| axon.snapshots.read.throughput.rate.oneMinuteRate            | average number of snapshot reads per second processed in the last minute                                                            |
| axon.snapshots.read.throughput.rate.fiveMinuteRate           | average number of snapshot reads per second processed in the five minutes                                                           |
| axon.snapshots.read.throughput.rate.fifteenMinuteRate        | average number of snapshot reads per second processed in the fifteen                                                                |
| axon.snapshots.read.duration                                 | Duration of read aggregate request                                                                                                  |
| axon.snapshots.read.error.count                              | Number of errors                                                                                                                    |
| axon.snapshots.read.saturation.active.count                  | Number of active aggregate read actions                                                                                             |

###  Scheduler metrics

| Metric name                                                  | Description                                                                                                                        |
|--------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| axon.tasks.saturation.scheduled.count | Number of scheduled tasks |
| axon.tasks.error.count | Number of errors executing tasks |
| axon.tasks.duration | Duration of task execution | 

### Application connection metrics

| Metric name                                   | Description                                |
|-----------------------------------------------|--------------------------------------------|
| axon.applications.duration.connection         | Duration of application connections        | 
| axon.applications.throughput.connect.count    | Number of application connect requests     |
| axon.applications.saturation.connected.count  | Number of applications currently connected | 
| axon.applications.throughput.disconnect.count | Number of application disconnect requests  |
| axon.authentication.error.count               | Number of authentication errors            |


### Deprecated metrics  <a id="application-level-metrics-deprecated"></a> 

The following metrics are deprecated in release 2023.2.0 and will be removed from Axon Server in release 2024.0.0: 

| Metric name                                        | Replaced by                                             | 
|----------------------------------------------------|---------------------------------------------------------| 
| axon.commands.count                                | axon.commands.throughput.count                          |
| axon.commands.rate.oneMinuteRate                   | axon.commands.throughput.rate.oneMinuteRate             |
| axon.commands.rate.fiveMinuteRate                  | axon.commands.throughput.rate.fiveMinuteRate            |
| axon.commands.rate.fifteenMinuteRate               | axon.commands.throughput.rate.fifteenMinuteRate         |
| axon.command                                       | axon.commands.duration                                  |
| axon.commands.active                               | axon.commands.saturation.active.count                   |
| axon.ApplicationCommandQueue.size                  | axon.commands.saturation.queued.count                   |
| axon.queries.count                                 | axon.queries.throughput.count                           |
| axon.queries.rate.oneMinuteRate                    | axon.queries.throughput.rate.oneMinuteRate              |
| axon.queries.rate.fiveMinuteRate                   | axon.queries.throughput.rate.fiveMinuteRate             |
| axon.queries.rate.fifteenMinuteRate                | axon.queries.throughput.rate.fifteenMinuteRate          |
| axon.query                                         | axon.queries.duration                                   |
| axon.queries.active                                | axon.queries.saturation.active.count                    |
| axon.ApplicationQueryQueue.size                    | axon.queries.saturation.queued.count                    |
| axon.event.count                                   | axon.events.append.throughput.count                     |
| axon.event.rate.oneMinuteRate                      | axon.events.append.throughput.rate.oneMinuteRate        |
| axon.event.rate.fiveMinuteRate                     | axon.events.append.throughput.rate.fiveMinuteRate       |
| axon.event.rate.fifteenMinuteRate                  | axon.events.append.throughput.rate.fifteenMinuteRate    |
| axon.snapshot.count                                | axon.snapshots.append.throughput.count                  |
| axon.snapshot.rate.oneMinuteRate                   | axon.snapshots.append.throughput.rate.oneMinuteRate     |
| axon.snapshot.rate.fiveMinuteRate                  | axon.snapshots.append.throughput.rate.fiveMinuteRate    |
| axon.snapshot.rate.fifteenMinuteRate               | axon.snapshots.append.throughput.rate.fifteenMinuteRate |
| axon.GlobalSubscriptionMetricRegistry.total        | axon.queries.subscriptionquery.throughput.total         |
| axon.GlobalSubscriptionMetricRegistry.updates      | axon.queries.subscriptionquery.throughput.updates       |
| axon.GlobalSubscriptionMetricRegistry.active       | axon.queries.subscriptionquery.saturation.active        |
| axon.QuerySubscriptionMetricRegistry.total         | axon.queries.subscriptionquery.throughput.total         |
| axon.QuerySubscriptionMetricRegistry.updates       | axon.queries.subscriptionquery.throughput.updates       |
| axon.QuerySubscriptionMetricRegistry.active        | axon.queries.subscriptionquery.saturation.active        |
| axon.ApplicationSubscriptionMetricRegistry.total   | axon.queries.subscriptionquery.throughput.total         |
| axon.ApplicationSubscriptionMetricRegistry.updates | axon.queries.subscriptionquery.throughput.updates       |
| axon.ApplicationSubscriptionMetricRegistry.active  | axon.queries.subscriptionquery.saturation.active        |

The deprecated metrics are still collected by default. To stop collecting the deprecated metrics set the property:
```
axoniq.axonserver.legacy-metrics-enabled=false
```


