# Axon Server metrics

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
| axon.events.read.aggregate.throughput.count                  | number of aggregates read since the start of the server                                                                             |
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


