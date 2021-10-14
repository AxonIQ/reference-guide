# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon JGroups Extension.

## Release 4.1

If cluster connection message came in quickly after starting the connection, a `NullPointerException` could be thrown.
This issue was resolved for release 4.1 [here](https://github.com/AxonFramework/extension-jgroups/issues/1).

## Release 4.0

We split off the JGroups logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
