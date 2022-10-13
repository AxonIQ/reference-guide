# Minor Releases

Any patch release made for an Axon project aims to resolve bugs.
This page provides an overview of patch releases for the Axon Tracing Extension.

## Release 4.6

### Release 4.6.1

#### Bug Fixes

- Move Flux tracing to utils class [#330](https://github.com/AxonFramework/extension-tracing/pull/330)

#### Dependency Upgrade

- Bump reactor.version from 3.4.23 to 3.4.24 [#328](https://github.com/AxonFramework/extension-tracing/pull/328)
- Bump axon.version from 4.6.0 to 4.6.1 [#327](https://github.com/AxonFramework/extension-tracing/pull/327)
- Bump kotlin.version from 1.7.10 to 1.7.20 [#326](https://github.com/AxonFramework/extension-tracing/pull/326)
- Bump slf4j.version from 2.0.2 to 2.0.3 [#325](https://github.com/AxonFramework/extension-tracing/pull/325)
- Bump actions/setup-java from 3.5.0 to 3.5.1 [#324](https://github.com/AxonFramework/extension-tracing/pull/324)
- Bump spring.boot.version from 2.7.3 to 2.7.4 [#323](https://github.com/AxonFramework/extension-tracing/pull/323)
- Bump slf4j.version from 2.0.1 to 2.0.2 [#322](https://github.com/AxonFramework/extension-tracing/pull/322)
- Bump junit-jupiter from 5.9.0 to 5.9.1 [#321](https://github.com/AxonFramework/extension-tracing/pull/321)
- Bump slf4j.version from 2.0.0 to 2.0.1 [#317](https://github.com/AxonFramework/extension-tracing/pull/317)

#### Contributors

We'd like to thank all the contributors who worked on this release!

- [@Morlack](https://github.com/Morlack)

## Release 4.5

### Release 4.5.2

* Bumped Axon Framework version used from 4.5. to 4.5.5 [here](https://github.com/AxonFramework/extension-tracing/commit/3cad35ce3fdba6e4398bb5471383685ecf2b686e) commit.
* An edge case was fixed where the `QueryGateway` could throws a `NullPointerException`. The fix for it was done [here](https://github.com/AxonFramework/extension-tracing/pull/218).


### Release 4.5.1

* With the release of Spring-Boot 2.6.0, the problem with circular dependencies came to light. To make it possible to use tracing-extension together with spring-boot 2.6.0, a circular dependency on our code was fixed as can be seen [here](https://github.com/AxonFramework/extension-tracing/commit/b4de5e3347568a7b5ca3c646ef96cdf4d1293f71).

## Release 4.3

### Release 4.3.1

* The tag for the query payload was wrong.
  We give credits to contributor `Sam-Kruglov` for noting the issue, which we resolved in [this](https://github.com/AxonFramework/extension-tracing/commit/72c8b15fec144c62fe6115d4993d60ab93ecee07) commit.

* In some Tracing UI's, camel-cased tag names showed up without the separation.
  This discrepancy is marked in issue [#45](https://github.com/AxonFramework/extension-tracing/issues/45) and resolved by using a dash notation instead of camel casing.
 