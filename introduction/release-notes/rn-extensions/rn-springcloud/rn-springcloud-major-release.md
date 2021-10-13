# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Spring Cloud Extension.

## Release 4.2

When using the [Kubernetes](https://spring.io/projects/spring-cloud-kubernetes) implementation of Spring Cloud the
`SpringCloudCommandRouter` would throw `NullPointerException`s.
This occurs because Spring Cloud Kubernetes does not support the `ServiceInstance`'s meta data field,
which the `SpringCloudCommandRouter` relies on.
[This](https://github.com/AxonFramework/extension-springcloud/pull/10) pull request introduced a null check to ensure
the null pointer would not be thrown again.

## Release 4.1

The `SpringCloudCommandRouter` failed to correctly connect to a Spring Cloud Discovery Service if the node did not contain any Command Handler methods.
This undesired behaviour was marked by user "travikk" and made more lenient under [this](https://github.com/AxonFramework/extension-springcloud/issues/1).

## Release 4.0

We split off the Spring Cloud logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
