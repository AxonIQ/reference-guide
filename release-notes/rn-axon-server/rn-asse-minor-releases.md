# Minor Releases Standard Edition

This page provides a dedicated overview of patch releases for the Axon Server (Standard Edition) releases

## Release 4.6

### Release 4.6.6

* Fix: memory leak due to a closed event processor keeping memory
* In the UI, the pagination settings for the tables are now kept in the current session
* Updated gRPC and Netty versions to avoid startup error on alpine linux
* Axon Server now checks for a configured token when it is started with access control enabled
* New logging statement to indicate that development mode is enabled

### Release 4.6.5

* Fix: update flow control library
* Fix: change health status for commands to only show warning when there are queued messages and no permits
* Updated audit logging for authentication failures


### Release 4.6.4

* Fix: change column name in event processor overview to "Active Segments"
* Fix: null pointer exception popup in dashboard

### Release 4.6.3

* Fix: reading aggregate events searches for older events when the last event sequence number
  is the same as the snapshot sequence number
* New property for index, axoniq.axonserver.event.segments-for-sequence-number-check, defines the number of segments
  that Axon Server will check for events on an aggregate when an event with sequence number 0
  is stored. The default value for this property is 10.
  For performance reasons, if you increase this property to a value higher than 100 it is recommended
  to also increase the axoniq.axonserver.event.max-bloom-filters-in-memory property.

### Release 4.6.2

* Fix: reading aggregate events hangs on JVM Error
* Fix: canceling an event store query through the gRPC interface does not close the stream
* Fix: event processor operations unavailable in the dashboard for applications using Axon Framework version before 4.5
* Fix: when sending two commands or queries with the same message identifier at the same time, one does not get completed

### Release 4.6.1

* Security update: updated control database settings

## Release 4.5

### Release 4.5.16

* Fix: reading aggregate events searches for older events when the last event sequence number
  is the same as the snapshot sequence number
* New property for index, axoniq.axonserver.event.segments-for-sequence-number-check, defines the number of segments
  that Axon Server will check for events on an aggregate when an event with sequence number 0
  is stored. The default value for this property is 10.
  For performance reasons, if you increase this property to a value higher than 100 it is recommended
  to also increase the axoniq.axonserver.event.max-bloom-filters-in-memory property.

### Release 4.5.15

* Fix: reading aggregate events hangs on JVM Error

### Release 4.5.14

* Security update: updated control database settings

### Release 4.5.13

* Reduced memory consumption during transactions
* Improved handling of out-of-memory exceptions
* Resolved a race condition in storing events that lead to delays in completing transactions

### Release 4.5.12

* Deprecated "/v1/backup/filenames" endpoint, use new endpoint /v1/backup/eventstore instead. The new endpoint returns all files
  to back up, given a last closed segment number, it also returns the currently last closed segment.

### Release 4.5.11

* Updated Spring Boot version to 2.5.12 to fix CVE-2022-22965

### Release 4.5.10

* Updated gRPC version from 1.42.0 to 1.42.2 to avoid CVE-2021-22569

### Release 4.5.9

* Updated gRPC and Netty versions
* Improved logging on client application disconnects
* Fix: missing/double icons on plugin page

### Release 4.5.8

* Update Felix to version 7.0.1 to support java 17
* Update JQuery to version 3.6.0
* Fix: incorrect login url when AS is invoked behind a reverse proxy
* Fix: NullPointerException in health check

### Release 4.5.7

* Fix: UI issues when running with another context root
* Fix: UI does not refresh the icons for event processor streams
* Fix: Balancing processors for a processing group containing special characters does not work from the UI
* Fix: Warning logged when a client closes an event stream while it is reading from old segments
* Remove timing metrics for commands/queries for clients no longer connected

### Release 4.5.6

* Fix: Memory leak in subscription query registrations

### Release 4.5.5

* Fix: Improved error handling and feedback when uploading invalid plugins
* Fix: Increase default settings for spring.servlet.multipart.max-request-size and spring.servlet.multipart.max-file-size to 25MB

### Release 4.5.4.1

* Fix: In case of timeout during query execution, AS sends a timeout error to the client before canceling the query.
* Fix: Close event store segment file when reading is complete

### Release 4.5.3

* Fix: Reset event store with multiple segments
* Fix: Regression in loading aggregate events performance
* Fix: Handle queries with same request type but different response type
* New metrics added:
  - file.bloom.open: counts the number of bloom filter segments opened since start
  - file.bloom.close: counts the number of bloom filter segments closed since start
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

## Release 4.4

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

## Release 4.3

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

## Release 4.2

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

## Release 4.1

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

