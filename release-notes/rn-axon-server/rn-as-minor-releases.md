# Minor Releases

This page aims to provide a dedicated overview of patch releases for the Axon Server releases

## Axon Server Standard Edition

## _Release 4.5_

### Release 4.5.2

* Improved performance for reading aggregates

  Axon Server is now reading events for an aggregate from multiple event store segments in parallel. The order in which
  Axon Server returns the events remains unchanged.

* Reduced memory usage for in-memory indexes

  Axon Server maintains index entries for the latest event store segment in-memory. The structure of this data has been
  changed to reduce the heap used by this index.

* Improvements in shutdown process
* Fix: Load balancing operations for processors should ignore stopped instances
* Fix: Stop reading events when query deadline expires

### Release 4.5.1

* Configurable strategy for aggregate events stream sequence validation (through property read-sequence-validation-strategy)
* Fix in UI for check for updates

## _Release 4.4_

### Release 4.4.12

* Fix: Load balancing operations for processors should ignore stopped instances
* Fix: Stop reading events when query deadline expires

### Release 4.4.11

* Configurable strategy for aggregate events stream sequence validation (through property read-sequence-validation-strategy)

### Release 4.4.10

* Fix for subscription queries in case of missing query handler

### Release 4.4.9

* Fix for concurrency issue in listing aggregates events during appending events for the same aggregate

### Release 4.4.8

* New metric to monitor query response times per query handler

### Release 4.4.7

* Improvement for subscription query: initial result are now provided by a single instance per component

### Release 4.4.6

* Fix for processor information showing information on disconnected applications
* Fix for issue with null expressions in ad-hoc queries
* Updated GRPC version to 1.34.0
* Added option to limit the number of commands/queries in progress

### Release 4.4.5

* Improved reporting of errors while initializing the event store
* Fix for NullPointerException when event processor status was sent to Axon Server before registration request
  was processed
* Improved handling of request processor status after an application disconnect

### Release 4.4.4

* Improved QueryService logging
* Added preserve event store option to delete context CLI command
* Fixed stream completed by the server in case of inactivity
* Hide upload license panel in SE
* Reduced number of open index files
* Fix for GetTokenAt operation

### Release 4.4.3

* Fix for connections not correctly registered
* Changed initialization sequence for event store to initialize completed segments first

### Release 4.4.2

* Offload expensive data-writing operations to a separate thread pool
* Fix for reading aggregates with older snapshots

### Release 4.4.1

* Reduced latency when Tracking live Events from a follower
* Improved handling of full queue to client
* Fix the refresh of the event processor status

## _Release 4.3_

### Release 4.3.6

* Fixed concurrency issue in subscribing/unsubscribing commands

### Release 4.3.5

* Fixed logging in IndexManager

### Release 4.3.4

* Reduced risk for contention when opening an index file
* Offload expensive data-fetching operations to separate thread pool
* Option to configure the way that index files are opened (memory mapped or file channel based)
* Limit the amount of commands/queries held in Axon Server waiting for the handlers to be ready to handle them, to avoid
  out of memory errors on Axon Server

### Release 4.3.3

* Fix for validation error starting up when there are multiple snapshot files (Standard Edition only)

### Release 4.3.2

* Fix for tracking event processor updates to websocket causing high CPU load in specific situation
* Reduced warnings in log file on clients disconnecting
* Fix for concurrency issue in sending heartbeat while client connects/disconnects

### Release 4.3.1

* Updated usage output in CLI
* Updated gRPC/Netty versions
* Prevent errors in log (sending ad-hoc result to client that has gone, sending heartbeat to client that has gone)

## _Release 4.2_

### Release 4.2.4

* Improved support for running management server on separate port

### Release 4.2.3

* Fix for pending queries with lost connection

### Release 4.2.2

* Added instruction acknowledgements
* Client applications heartbeat support
* Cleaned-up logging
* Fix for specific error while reading aggregate
* Optional heartbeat between Axon Server and Axon Framework clients

### Release 4.2.1

* Fixes required for enterprise edition only

## _Release 4.1_

### Release 4.1.7

* Use info endpoint to retrieve version number and product name
* Reset reserved sequence numbers for aggregate when storing the event failed

### Release 4.1.6

* Added operation to set cached version numbers for aggregates

### Release 4.1.5

* Fix for authorization path mapping and improvements for rest access control
* Improvements in release procedure for docker images
* Fix for subscription query memory leak
* Improvements in error reporting in case of disconnected applications
* Improvements in detection of insufficient disk space

### Release 4.1.4

* Fix for appendEvent with no events in stream

### Release 4.1.3

* CLI commands now can be performed locally without token.

### Release 4.1.2

* Status displayed for tracking event processors fixed when segments are running in different applications
* Tracking event processors are updated in separate thread
* Logging does not show application data anymore
* Changed some gRPC error codes returned to avoid clients to disconnect when no command handler found for a command

### Release 4.1.1

* Sources now available in public GitHub repository
* Merge tracking event processor not always available when it should
* Logging changes
* GRPC version update

## Axon Server Enterprise Edition

## _Release 4.5_

### Release 4.5.3

* Improved performance for reading aggregates
  
  Axon Server is now reading events for an aggregate from multiple event store segments in parallel. The order in which 
  Axon Server returns the events remains unchanged. 

* Reduced memory usage for in-memory indexes
  
  Axon Server maintains index entries for the latest event store segment in-memory. The structure of this data has been 
  changed to reduce the heap used by this index.    

* Reduced number of files kept open as memory-mapped files
  
  Axon Server now only keeps a (configurable) number of event files open as memory-mapped files. Older files will be 
  opened and closed when needed. The number of event store segments that are opened as memory mapped files can be 
  configured through the property: axoniq.axonserver.event.memory-mapped-segments

* Initialize event stores asynchronously on startup
  
  When Axon Server needs to recreate indexes for a context on startup, it can take some time to complete. This 
  change makes the initialization of the context asynchronous, so that if one context takes a long time to initialize, 
  other replication groups are already available. This also makes the HTTP endpoint available sooner, so liveliness 
  checks succeed faster.

* Improvements in shutdown process
* Fix: Load balancing operations for processors should ignore stopped instances
* Fix: Stop reading events when query deadline expires
* Fix: Disparities in Context Leaders
* Dependency update: updated xstream version used to 1.4.17

### Release 4.5.2

* Configurable strategy for aggregate events stream sequence validation (through property read-sequence-validation-strategy)
* Fix UI check for updates

### Release 4.5.1

* Fix for initialization error for the transaction log
* Fix for node not able to replicate after install snapshot procedure
* Fix for concurrency issue in creating schedulers for auto-loadbalancing

## _Release 4.4_

### Release 4.4.15

* Fix: Load balancing operations for processors should ignore stopped instances
* Fix: Stop reading events when query deadline expires
* Dependency update: updated xstream version used to 1.4.17

### Release 4.4.14

* Configurable strategy for aggregate events stream sequence validation (through property read-sequence-validation-strategy)
* Fix for initialization error for the transaction log
* Fix for node not able to replicate after install snapshot procedure
* Fix for concurrency issue in creating schedulers for auto load balancing

### Release 4.4.13

* Fix for race condition during JumpSkipIndex update
* Fix for memory leak during peer node registration in Raft leader

### Release 4.4.12

* Fix for subscription queries in case of missing query handler

### Release 4.4.11

* Fix for concurrency issue in listing aggregates events during appending events for the same aggregate

### Release 4.4.10

* Load balancing strategy for queries is now configurable. Metrics based by default, to configure round-robin set property
  axoniq.axonserver.query-handler-selector=round-robin
* Improvement on metrics based load balancing of queries, now giving higher probabilities based on the response time
  for the handler as recorded on the current axon server node
* Fix for scheduler not rescheduling all events after a leader change

### Release 4.4.9

* Improvement for subscription query: initial result are now provided by a single instance per component

### Release 4.4.8

* Fix for log compaction never performed if the backup is taken more often than each hour
* Fix in release notes for version 4.4.7
* Fix for processor information showing information on disconnected applications
* Fix for issue with null expressions in ad-hoc queries
* Updated GRPC version to 1.34.0
* Added option to limit the number of commands/queries in progress per context

### Release 4.4.7

* Fix for access control issue when updating an application
* Fix for global index files remaining after delete context
* Fix for commit index not updated correctly in specific cases
* Fix for warning messages on opening index that should have been debug messages
* Fix for invalid previous term/index in replication heartbeat messages causing some overhead in communication
* Fix for missing portnumber in cluster template export
* Fix for commands being sent to wrong context if the only handler is in another context
* New property "axoniq.axonserver.enterprise.default-index-type" to specify the index type for new contexts. Default
  value is JUMP_SKIP_INDEX, use BLOOM_FILTER_INDEX for the pre-4.4 index type

### Release 4.4.6

* Improved QueryService logging
* Added preserve event store option to delete context CLI command
* Fixed stream completed by the server in case of inactivity
* Hide upload license panel in SE
* Reduced number of open index files
* Fix for GetTokenAt operation
* Improved feedback on license upload errors
* Fix timing issue in leader change potentially causing duplicate events
* New REST endpoint to download a diagnostics zip file

### Release 4.4.5

* Fix for connections not correctly registered
* Changed initialization sequence for event store to initialize completed segments first
* Changed order of files in the backup endpoint for contexts with a jump skip index to list the global index files
  before the segment index files
* Fix timing issue when a follower sends new events to a new leader before it is fully initialized
* Improved logging and error handling for log compaction task

### Release 4.4.4

* Fix for initializing jump skip index when it has more than 1 segment
* Improved error handling for problems creating a context and storing events/snapshots
* Offload expensive data-writing operations to separate thread pool
* Fix for reading aggregates with older snapshots

### Release 4.4.3

* Fix race condition in queries and commands handlers unsubscription during reconnection
* Fix pre-vote election with active backup nodes
* Axon Server SE improvements from 4.4 to 4.4.1
* Fixed issue causing added latency while Tracking live Events from a follower node
* Fix the event processor status refresh process

### Release 4.4.2

* Fix for downloading and starting with cluster templates
* Renamed field replicationsGroups to replicationGroup in cluster template
* Fix storing entries when a context is created with pre-existing event store
* Fix query from dashboard when query is executed from admin node and the target context is not defined on this admin node
* Fix for timing issue in sharing metrics between Axon Server nodes causing exception during delete context

### Release 4.4.1

* Fix for migration of contexts created before version 4.3

## _Release 4.3_

### Release 4.3.7

* Fixed concurrency issue in subscribing/unsubscribing commands

### Release 4.3.6

* Do not override log entries in RAFT log that already have been committed

### Release 4.3.5

* Fix for scheduling of tasks in clustering module
* Do not override log entries in RAFT log that already have been committed
* Fixed logging in IndexManager

### Release 4.3.4

* Reduced risk for contention when opening an index file
* Offload expensive data-fetching operations to separate thread pool
* Option to configure the way that index files are opened \(memory mapped or file channel based\)
* Limit the amount of commands/queries held in Axon Server waiting for the handlers to be ready to handle them, to avoid

  out of memory errors on Axon Server

* Fix for high number of cluster-request threads being created
* Fix for timing issue in delete context. This could leave the context existing on one of the member nodes
* Fix RAFT bug: configuration changes are not allowed before an entry has been committed in the current term.

New configuration properties added for Axon Server:

_axoniq.axonserver.data-fetcher-threads_=24 \(number of threads that are allocated for doing longer running operations on the event store\)

_axoniq.axonserver.command-queue-capacity-per-client_=10000 \(number of command requests for a specific command handling client that Axon Server will cache waiting for permits\)

_axoniq.axonserver.query-queue-capacity-per-client_=10000 \(number of query requests for a specific query handling client that Axon Server will cache waiting for permits\)

_axoniq.axonserver.replication.use-mmap-index_=null \(by default, AxonServer will determine whether to use memory mapped indexes for replication logs based on operating system and java version, in rare cases it may be useful to override the default\)

_axoniq.axonserver.replication.force-clean-mmap-index_=null \(option to forcefully close unused memory mapped files instead of leaving the garbage collector do this, by default, AxonServer will determine this based on operating system and java version, in rare cases it may be useful to override the default\)

_axoniq.axonserver.event.use-mmap-index_=null \(by default, AxonServer will determine whether to use memory mapped indexes for event files in the event store based on operating system and java version, in rare cases it may be useful to override the default\)

_axoniq.axonserver.event.force-clean-mmap-index_=null \(option to forcefully close unused memory mapped files instead of leaving the garbage collector do this, by default, AxonServer will determine this based on operating system and java version, in rare cases it may be useful to override the default\)

_axoniq.axonserver.snapshot.use-mmap-index_=null \(by default, AxonServer will determine whether to use memory mapped indexes for snapshot files in the event store based on operating system and java version, in rare cases it may be useful to override the default\)

_axoniq.axonserver.snapshot.force-clean-mmap-index_=null \(option to forcefully close unused memory mapped files instead of leaving the garbage collector do this, by default, AxonServer will determine this based on operating system and java version, in rare cases it may be useful to override the default\)

_**Configuration properties default values changed:**_

_axoniq.axonserver.cluster-executor-thread-count_=4 \(reduced from 8, configures the number of threads used to handle requests from other axon server nodes, reduced as effective processing of the requests if offloaded to other threads\)

_axoniq.axonserver.executor-thread-count_=4 \(reduced from 8, configures the number of threads used to handle requests from clients, reduced as effective processing of the requests if offloaded to other threads\)

### Release 4.3.3

* Fix for race condition during rollback of transaction log

### Release 4.3.2

* Fix for tracking event processor updates to websocket causing high CPU load in specific situation
* Reduced warnings in log file on clients disconnecting
* Fix for concurrency issue in sending heartbeat while client connects/disconnects

### Release 4.3.1

* Updated usage output in CLI
* Updated gRPC/Netty versions
* Prevent errors in log \(sending ad-hoc result to client that has gone, sending heartbeat to client that has gone\)
* Fix in CLI, application list failed
* Removed retry option from H2 controldb URL
* Fixed Axon Server keeping incorrect application references on restart of Axon Server node where application was connected
* Fixed race condition when two Axon Server nodes are trying to become member of the same context at the same time
* Fix for some REST reads were able with invalid token
* Clean up of server shutdown process
* Fixed error when adding a non-existing node to a context

## _Release 4.2_

### Release 4.2.6

* Rollback in log entry could cause cluster member to transition to fatal state when there were writes pending

### Release 4.2.5

* Fix for start-up issue when management.server.port is set

### Release 4.2.4

* Added instruction acknowledgements
* Client applications heartbeat support
* Cleaned-up logging
* Fix for specific error while reading aggregate
* Optional heartbeat between Axon Server and Axon Framework clients

### Release 4.2.3

* Fix for issue on cancelling queries on lost connection
* Introduced acknowlegments on instructions between Axon Server and clients

### Release 4.2.2

* Fix for continuously checking tracking event processor status
* Fix for issue in access control: combining roles for all contexts and single context

### Release 4.2.1

* Fix for issue in access control: applications with access to multiple contexts are not always correctly validated
* Fix for UI issue: after deleting an application the buttons no longer work
* On auto-load-balance of a event processor group only request the current status from applications with this processor group

## _Release 4.1_

### Release 4.1.9

* Fix for situation where election takes too long to complete causes 2 leaders
* Wait for leader when event store requests arrive at Axon Server during leader election. New configuration property

  axoniq.axonserver.replication.wait-for-leader-timeout to change this wait time. Default is 2\*max election timeout.

### Release 4.1.8

* Fix for unexpected aggregate sequence exception.
* Fix for wrong version number in dashboard.

### Release 4.1.7

* Fix for timing issue when new events arrived at new leader before leader was fully initialized
* Fix for configuration not always applied on all nodes
* Fix for OutOfDirectMemory issue when installing snapshot

### Release 4.1.6

* Fix for unexpected leader change on adding 3rd node to context
* Fix for replication entry log compaction

### Release 4.1.5

* Fix for mismatch in connected applications after rebalance
* Fixes in access control check on REST endpoints for events/snapshot endpoints and renew application token
* Improved error handling in replication
* Fixed occasional timing issue in create context
* Improved recovery options after full data loss on node
* Fix for authorization path mapping
* Fix for subscription query memory leak
* Improvements in error reporting in case of disconnected applications
* Improvements in detection of insufficient disk space

### Release 4.1.4

* Fix on RAFT Leader detection
* Avoided multiple concurrent creation of the same context
* Fix for appendEvent with no events in stream
* Stop log cleaning task when context is deleted or node is removed from context
* Fix for NullPointerException on lost connection

### Release 4.1.3

* Improved recovery of cluster after disk failure
* Fix for updating configuration during install snapshot
* CLI commands now can be performed locally without token.

### Release 4.1.2

* Improvements in replication to nodes that have been down for a long time
* Tracking event processor auto-loadbalancing fixes
* Status displayed for tracking event processors fixed when segments are running in different applications
* Tracking event processors are updated in separate thread
* Logging does not show application data anymore
* Fixed Axon Dashboard list of cluster nodes missing hostname and ports
* Changed some gRPC error codes returned to avoid clients to disconnect
* CLI commands init-cluster/register-node/add-node-to-context are now idempotent
* Register node returns faster \(no longer waits for synchronization to new node\)
* Reduced risk of re-election if a node restarts
* Fixed occasional NullPointerException when client connects to a newly registered Axon Server node
* Fixed incorrect leader returned in context API and multiple leaders warning in log

### Release 4.1.1

* Default controldb connection settings changed
* gRPC version update
* Register node no longer needs to be sent to leader of \_admin context
* Merge tracking event processor not always available when it should
* Logging changes
* Fix for queries timeout
* Fix for replication with large messages
* Added axonserver-cli.jar to release package \(axoniq-cli.jar is deprecated\)

