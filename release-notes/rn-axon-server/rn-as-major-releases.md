# Major Releases

## Release 4.5

Standard Edition and Enterprise Edition

New features:
- Support for customer-defined plugins to add custom actions to adding/reading events and snapshots and executing commands and (subscription queries)
  For more information see [plugins](/axon-server/administration/plugins).
- Search snapshots in Axon Dashboard

Enhancements:
- Flow control for reading aggregates
- Updates in user permissions now have immediate effect (even if the user is logged in)
- Logging of illegal access to Axon Server gRPC services
- Improved monitoring of available disk space (see [actuator-endpoints](../../axon-server/administration/monitoring/actuator-endpoints))
- List of used 3rd party libraries available from Dashboard
- Axon Dashboard checks for Axon Server version updates

Dependency updates:
- updated gRPC and Netty versions
- updated Spring Boot version
- updated Swagger version

Notes:
- Due to the update of the Spring Boot version there are some minor changes to the output of the /actuator/health endpoint.
  This endpoint now uses the element "components" instead of "details" to output the health per subcategory.

- The swagger endpoint has changed from /swagger-ui.html to /swagger-ui/.

Standard Edition only

Bug fixes:
- Read aggregate snapshots from closed segments fixed

Enterprise Edition only

New features:
- Authentication via Google OAuth (via extension)
- Authentication and authorization with LDAP
- New role VIEW_CONFIGURATION allowing user read only access to the configuration of Axon Server in the Dashboard

Enhancements:
- Option to use a separate SSL key file for communication between Axon Server nodes (see property internal-private-key-file in [configuration](../../axon-server/administration/admin-configuration/configuration))
- License status added to health page (see [actuator-endpoints](../../axon-server/administration/monitoring/actuator-endpoints))
- Migration tool now supports migrating from a MongoDB event store to Axon Server

Bug fixes:
- memory leak in the replication group leader could cause elections
- incorrect version number shown in login page and error page in Dashboard
- invalid number of active subscription queries displayed in Dashboard
- exception initializing a context for the replication group leader on restart after unclean shutdown

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

