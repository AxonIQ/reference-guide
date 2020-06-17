# Axon Framework Extensions

## Release Notes

### JGroups 4.2

As marked under [this](https://github.com/AxonFramework/extension-jgroups/issues/4) issue, the command callback will not be called if the connection between JGroups peers dies whilst a command is in transit. Credits go to "sgrimm-sg" for filing the issue and solving [it](https://github.com/AxonFramework/extension-jgroups/pull/5).

### JGroups 4.1

If cluster connection message came in quickly after starting the connection, a `NullPointerException` could be thrown. This issue was resolved for release 4.1 [here](https://github.com/AxonFramework/extension-jgroups/issues/1).

### Mongo 4.0.1

The Mongo extension incorrectly used the content type instead of the tokens type upon storing a serialized token. The issue was marked and resolved under [\#1](https://github.com/AxonFramework/extension-mongo/issues/1).

### Spring Cloud 4.2

When using the [Kubernetes](https://spring.io/projects/spring-cloud-kubernetes) implementation of Spring Cloud the `SpringCloudCommandRouter` would throw `NullPointerException`s. This occurs because Spring Cloud Kubernetes does not support the `ServiceInstance`'s meta data field, which the `SpringCloudCommandRouter` relies on. [This](https://github.com/AxonFramework/extension-springcloud/pull/10) pull request introduced a null check to ensure the null pointer would not be thrown again.

### Spring Cloud 4.1

The `SpringCloudCommandRouter` failed to correctly connect to a Spring Cloud Discovery Service if the node did not contain any Command Handler methods. This undesired behaviour was marked by user "travikk" and made more lenient under [this](https://github.com/AxonFramework/extension-springcloud/issues/1).

