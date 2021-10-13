# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Spring Cloud Extension.

## Release 4.1

The `SpringCloudCommandRouter` failed to correctly connect to a Spring Cloud Discovery Service if the node did not contain any Command Handler methods.
This undesired behaviour was marked by user "travikk" and made more lenient under [this](https://github.com/AxonFramework/extension-springcloud/issues/1).

## Release 4.0

We split off the Spring Cloud logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
