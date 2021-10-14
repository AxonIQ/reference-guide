# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Spring Cloud Extension.

## Release 4.4

This release has seen a bunch of adjustments towards the Spring Cloud extension, which can be grouped into two categories.
Firstly, the introduction of the `CapabilityDiscoveryMode` and secondly the automatic process added to this project:

* Introduced GitHub Actions to build, test and push snapshots of the Spring Cloud Extension, as has been marked in pull request [#68](https://github.com/AxonFramework/extension-springcloud/pull/68).
* Dependabot was introduced, ensuring all versions will be as up-to-date as possible.
* JUnit4 has been removed entirely, in favor of JUnit 5.
* The `CapabilityDiscoveryMode` mode has been introduced through issue [#23](https://github.com/AxonFramework/extension-springcloud/issues/23).
  This approach allows for more flexibility when it comes to defining how command routing information should be retrieved and shared.

For a full list of all the changes, please check the [release notes](https://github.com/AxonFramework/extension-springcloud/releases/tag/axon-springcloud-4.4).

## Release 4.3

* Issue [#14](https://github.com/AxonFramework/extension-springcloud/pull/14) implements the `CommandBusConnector#localSegment`.
  Axon Framework introduces this method in release 4.3 (and issue [#874](https://github.com/AxonFramework/AxonFramework/issues/874)) to ensure the usage of the `DisruptorCommandBus`'s repository is followed when distributing the `CommandBus`.

* We introduced a graceful start-up and shutdown solution in Axon Framework release 4.3 for all infrastructure components.
  Issue [#15](https://github.com/AxonFramework/extension-springcloud/pull/15) ensures the `SpringHttpCommandBusConnector` complies with this style too.

For a complete list of all changes, see [this](https://github.com/AxonFramework/extension-springcloud/issues?q=is%3Aclosed+milestone%3A%22Release+4.3%22) page.

## Release 4.2

When using the [Kubernetes](https://spring.io/projects/spring-cloud-kubernetes) implementation of Spring Cloud the
`SpringCloudCommandRouter` would throw `NullPointerException`s.
This occurs because Spring Cloud Kubernetes does not support the `ServiceInstance`'s metadata field,
which the `SpringCloudCommandRouter` relies on.
[This](https://github.com/AxonFramework/extension-springcloud/pull/10) pull request introduced a null check to ensure
the null pointer would not be thrown again.

## Release 4.1

The `SpringCloudCommandRouter` failed to correctly connect to a Spring Cloud Discovery Service if the node did not contain any Command Handler methods.
This undesired behaviour was marked by user "travikk" and made more lenient under [this](https://github.com/AxonFramework/extension-springcloud/issues/1).

## Release 4.0

We split off the Spring Cloud logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
