# Major Releases

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

