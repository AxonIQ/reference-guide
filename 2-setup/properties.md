# 3.2 Properties

## Configuration Properties

The following table gives a complete list of all the configuration properties available for the Axon Server.

| Name | Default value | Type | Description |
| :--- | :--- | :--- | :--- |
| server.port | 8024 | Integer | The Http Server port |
| axoniq.axonserver.name |  | String | Node name of this Axon Server node, if not set defaults to the hostname. |
| axoniq.axonserver.port | 8124 | Integer | gRPC port for Axon Server |
| axoniq.axonserver.internal-port | 8224 | Integer | gRPC port for communication between Axon Server nodes |
| axoniq.axonserver.hostname |  | String | Hostname of this node as communicated to clients, defaults to the result of hostname command |
| axoniq.axonserver.internal-hostname |  | String | Hostname as communicated to other nodes of the cluster. Defaults to hostname. |
| axoniq.axonserver.domain |  | String | Domain of this node as communicated to clients. Optional, if set will be appended to the hostname in communication with clients. |
| axoniq.axonserver.internal-domain |  | String | Domain as communicated to other nodes of the cluster. Optional, if not set, it will use the domain value. |
| axoniq.axonserver.accesscontrol.enabled | false | Boolean | Access control active |
| axoniq.axonserver.accesscontrol.cache-ttl | 30000 | Long | Milliseconds that authenticated tokens will be cached |
| axoniq.axonserver.accesscontrol.internal-token |  | String | Token to add to Axon Server internal cluster message |
| axoniq.axonserver.accesscontrol.token |  | String | Token expected from client requests \[Developer edition only\] |
| axoniq.axonserver.cluster.enabled | false | Boolean | Cluster enabled |
| axoniq.axonserver.cluster.connection-check-delay | 1000 | Long | Delay before the first run of the connection checker \(in ms.\) |
| axoniq.axonserver.cluster.connection-check-interval | 1000 | Long | Interval between each run of the connection checker \(in ms.\) |
| axoniq.axonserver.cluster.rebalance-delay | 7 | Long | Delay before the first run of the rebalancer \(in seconds\) \[Enterprise edition only\] |
| axoniq.axonserver.cluster.rebalance-interval | 15 | Long | Interval between each run of the rebalancer \(in seconds\) \[Enterprise edition only\] |
| axoniq.axonserver.command-flow-control.initial-permits | 10000 | Long | Initial number of permits granted in communication between Axon Server nodes. |
| axoniq.axonserver.command-flow-control.new-permits | 10000 | Long | Additional number of permits granted in communication between Axon Server nodes. |
| axoniq.axonserver.command-flow-control.threshold | 1000 | Long | Threshold at which the node will send another grant of newPermits to the connected platform node. |
| axoniq.axonserver.query-flow-control.initial-permits | 10000 | Long | Initial number of permits granted in communication between Axon Server nodes. |
| axoniq.axonserver.query-flow-control.new-permits | 10000 | Long | Additional number of permits granted in communication between Axon Server nodes. |
| axoniq.axonserver.query-flow-control.threshold | 1000 | Long | Threshold at which the node will send another grant of newPermits to the connected platform node. |
| axoniq.axonserver.ssl.enabled | false | Boolean | SSL enabled for gRPC servers |
| axoniq.axonserver.ssl.cert-chain-file |  | String |  |
| axoniq.axonserver.ssl.internal-cert-chain-file |  | String |  |
| axoniq.axonserver.ssl.private-key-file |  | String |  |
| axoniq.axonserver.event.storage |  | String | Location for the event data |
| axoniq.axonserver.snapshot.storage |  | String | Location for the snapshot data, defaults to the same as the event data. |
| axoniq.axonserver.controldb-path | . | String | Path to the Axon Server controlDB, note that this has to be an absolute path or start with '.' |
| axoniq.axonserver.controldb-backup-location | . | Path | Directory where backups of the controlDB are created |
| axoniq.axonserver.max-message-size | 0 | int | Maximum allowed GRPC inbound message size, 0 keeps GRPC default value |

## Client Configuration Properties

The following table gives a complete list of all the configuration properties available for the Axon Server Client.

| Name | Default value | Type | Description |
| :--- | :--- | :--- | :--- |
| axon.axonserver.servers | localhost | String | Comma separated list of Axon Server servers |
| axon.axonserver.ssl-enabled | false | Boolean | Use SSL for connection to Axon Server |
| axon.axonserver.cert-file |  | String | Path to certificate file to use for SSL |
| axon.axonserver.token |  | String | Authentication token sent with each request. Only if Axon Server is running in Authentication mode. |
| axon.axonserver.client-name |  | String | Client name as communicated to Axon Server. If not set, defaults to hostname:processId. |
| axon.axonserver.component-name |  | String | Component name, all instances of the same application must have the same component name. Defaults to Spring application name. |
| axon.axonserver.initial-nr-of-permits | 100000 | Long | Initial number of permits granted in communication with Axon Server. |
| axon.axonserver.new-permits-threshold | 1000 | Long | Threshold at which the node client send another grant of newPermits to the connected platform node. |
| axon.axonserver.nr-of-new-permits | 90000 | Long | Additional number of permits granted in communication with Axon Server nodes. |
| axon.axonserver.command-threads | 10 | Integer | Number of threads processing commands |
| axon.axonserver.query-threads | 10 | Integer | Number of threads processing queries |
| axon.axonserver.context |  | String | Axon Server context used by the application \(Enterprise Edition only\) |

