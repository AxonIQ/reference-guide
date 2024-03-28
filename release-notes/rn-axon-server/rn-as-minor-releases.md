# Minor Releases

This page provides a dedicated overview of patch releases for Axon Server. For information about the
older releases check either [Axon Server Enterprise Edition](rn-asee-minor-releases.md) or [Axon Server Standard Edition](rn-asse-minor-releases.md).

## Release 2023.2

### Release 2023.2.3

Bug fix:

- Increasing number of threads on the running Axon Server nodes when one node in the cluster is down.

### Release 2023.2.2

Bug fixes:

- Fix for an error handling subscription query responses during the upgrade from a version before 2023.2.0 to 2023.2.0 or 2023.2.1.
- Improved readiness probe to return 200 (OK) once the communication services are ready and the replication groups are completely initialized.
  The endpoint for the new readiness probe is /actuator/health/readiness.

### Release 2023.2.1

#### Bug Fixes

This release contains fixes for the following issues:

- TLS communication between Axon Server nodes cannot validate trusted certificates when there is no trust manager file configured
- Deleting a context does not delete all its metrics


## Release 2023.1

### Release 2023.1.2

#### Bug Fixes

This release contains fixes for the following issues:
- Metrics no longer collected when an application reconnects to Axon Server

### Release 2023.1.1

#### New Features and Enhancements:

_Initialize standalone_ 

To simplify initialization of Axon Server, it now supports a new property "axoniq.axonserver.standalone=true". When this property is set on a clean Axon Server instance it initializes the server with a "default" context.

_Development mode_

Fixed the option to reset the event store from the UI (in development mode). This option now also works in an
Axon Server cluster.

_LDAP extension update_

The new version of the LDAP extension supports configuration of a trust manager file. The location of the file
can be specified through the property "axoniq.axonserver.enterprise.ldap.trust-manager-file".

#### Bug Fixes

This release contains fixes for the following issues:
- Validation of tiered storage properties when not using the UI
- Race condition while writing to the global index
- Limitation on the number of requests per context fails if there are timed out requests

