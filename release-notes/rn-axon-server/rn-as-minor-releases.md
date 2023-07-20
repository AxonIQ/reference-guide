# Minor Releases

This page provides a dedicated overview of patch releases for Axon Server. For information about the
older releases check either [Axon Server Enterprise Edition](rn-asse-minor-releases.md) or [Axon Server Standard Edition](rn-asse-minor-releases.md).

## Release 2023.1

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

