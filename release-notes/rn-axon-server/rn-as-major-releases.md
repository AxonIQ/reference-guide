# Major Releases

## Release 4.4

Standard Edition and Enterprise Edition

* Axon Server can now act as an event scheduler
* Tag-based routing of commands and queries
* Support fom token store identifiers to identify which tracking event processors share a token store

Enterprise Edition only

* Tracking event processors can now read from any primary node
* Reading aggregates will read events from followers and only request the leader for latest events
* Introduction of replication groups containing one or more contexts to reduce overhead in replication process
* New index type to improve speed in reading aggregates
* Axon Server Enterprise can now start without a license and has the option to upload licenses to the cluster
* Cluster templates to initialize a cluster including replication groups, contexts, users and applications based on a
    template
* Multi-tier storage to keep only recent data at primary nodes
* Some storage properties can now be set at the context level

## Release 4.3

* Introduced new roles for nodes in context \(ACTIVE\_BACKUP, PASSIVE\_BACKUP and MESSAGING\_ONLY\)
* Improved support for running in containers
* New option to configure cluster information in configuration files, to automatically build cluster at startup
* Support load factor for commands
* Support for moving Axon Server cluster to other nodes
* Option to remove a node from a context without deleting the event store on that node
* Separate audit log for configuration changes
* Changed metrics to use common names and tags

## Release 4.2

* Delete leader from group is now possible.
* Removing a context from a node, deletes all data \(including event data\) on that node.
* Deleting a context removes all data \(including event data\).
* Blacklisting event types for applications that cannot handle these events \(requires AxonFramework v4.2.\)
* Expose tracking event processor position and status  \(requires AxonFramework v4.2.\)
* Clients can provide preferences on which node to connect to  \(requires AxonFramework v4.2.\)
* New access control roles
* Reduced leader changes by new pre-vote phase
* Improved health page

## Release 4.1

* Support for Split/Merge of tracking event processors through user interface
* Introduction of \_admin context for all cluster management operations
* Updated process for getting started \(see above\)
* Replication of data and configuration between nodes in the cluster is now based on transaction log replication.

  You will see new files created on AxonServer nodes in the log directory \(axoniq.axonserver.replication.log-storage-folder\).

  Do not delete those files manually!

* Default setting for health endpoint \(/actuator/heath\) has changed to show more details.
* Change in TLS configuration for communication between AxonServer nodes \(new property axoniq.axonserver.ssl.internal-trust-manager-file\)

