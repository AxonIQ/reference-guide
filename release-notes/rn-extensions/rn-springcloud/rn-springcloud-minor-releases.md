# Minor Releases

Any patch release made for an Axon project aims to resolve bugs.
This page provides an overview of patch releases for the Axon Spring Cloud Extension.

## Release 4.9

### Release 4.9.1

#### Bug Fixes

- [#327] Default local `ServiceInstance` to a fixed `URI` i.o. `null` [#330](https://github.com/AxonFramework/extension-springcloud/pull/330)
- Instance without Command Handlers cannot be discovered [#327](https://github.com/AxonFramework/extension-springcloud/issues/327)

#### Contributors

We'd like to thank all the contributors who worked on this release!

- [@smcvb](https://github.com/smcvb)

## Release 4.3

### Release 4.3.1

The most recent release of [Spring Cloud Eureka](https://cloud.spring.io/spring-cloud-netflix/reference/html/) showed this extension fallback approach to be the only way to discover instances.
Issue [#17](https://github.com/AxonFramework/extension-springcloud/issues/17) introduces functionality to enforce the usages of the fallback approach. 
