# Minor Releases

This page provides a dedicated overview of patch releases for the Axon Server releases

## _Release 4.5_

### Release 4.5.3

* Fix: Reset event store with multiple segments
* Fix: Regression in loading aggregate events performance
* Fix: Handle queries with same request type but different response type
* New metrics added:
  - file.bloom.open: counts the number of bloom filter segments opened since start
  - file.bloom.close: counts the number of bloom filter segments closed since start
  - file.bloom.open: counts the number of bloom filter segments opened since start
  - file.segment.open: counts the number of event store segments opened since start
  - local.aggregate.segments: monitors the number of segments that were accessed for reading aggregate event requests

Notes:
- Default value for configuration property axoniq.axonserver.event.events-per-segment-prefetch is decreased from 50 to 10.

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

