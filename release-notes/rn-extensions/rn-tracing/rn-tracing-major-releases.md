# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Tracing Extension.

## Release 4.6

If you're curious about the dependency upgrades made in this release we refer to [this](https://github.com/AxonFramework/extension-tracing/releases/tag/axon-tracing-4.6.0) page.

### Features

- Adds support for Streaming Query [#280](https://github.com/AxonFramework/extension-tracing/pull/280)
- TracingQueryGateway does not implement streamingQuery [#279](https://github.com/AxonFramework/extension-tracing/issues/279)

### Enhancements

- Change how Sonar is invoked for GHAs [#228](https://github.com/AxonFramework/extension-tracing/pull/228)
- Splitted builds into pr and not pr, also added ghactions to dependabot [#155](https://github.com/AxonFramework/extension-tracing/pull/155)
- Validate OpenTelemetry API [#90](https://github.com/AxonFramework/extension-tracing/issues/90)

### Bug Fixes

- Fix sonar build in PRs [#273](https://github.com/AxonFramework/extension-tracing/pull/273)
- Correct application run command to match jar name [#205](https://github.com/AxonFramework/extension-tracing/pull/205)
- Added project reactor as provided scope [#144](https://github.com/AxonFramework/extension-tracing/pull/144)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@smcvb](https://github.com/smcvb)
- [@Morlack](https://github.com/Morlack)
- [@pasquatch913](https://github.com/pasquatch913)
- [@lfgcampos](https://github.com/lfgcampos)
- [@schananas](https://github.com/schananas)


## Release 4.5

* Contributor `aupodogov` provided an optimization in the `OpenTraceHandlerInterceptor`, by replace an `orElse` for `orElseGet`, in pull request [#103](https://github.com/AxonFramework/extension-tracing/pull/103).
  
* We introduced a sample module that shows how to use this extension.
  You can find the module [here](https://github.com/AxonFramework/extension-tracing/tree/master/tracing-axon-example).

* Spring Boot ordering sometimes didn't wire the tracing gateways.
  To solve this, we enforced the ordering through an `AutoConfigureBefore` annotation on the `TracingAutoConfiguration`.
  We marked this under issue [#105](https://github.com/AxonFramework/extension-tracing/issues/105) and solved it in [this](https://github.com/AxonFramework/extension-tracing/pull/106) pull request.

We refer to the 4.5 [release notes](https://github.com/AxonFramework/extension-tracing/releases/tag/axon-tracing-4.5) for a complete list of all changes.

## Release 4.4

* Pull request [#78](https://github.com/AxonFramework/extension-tracing/pull/78) introduces an enhancement for users of this extension.
  With the `MessageTagBuilderService` they have more configuration options for selecting the desired tags per message type.

* When using Spring Boot, the gateway bean names this extension builds clashed with Axon Frameworks default `CommandGateway` and `QueryGateway` beans.
  Contributor `guilhermeblanco` marked this in issue [#79](https://github.com/AxonFramework/extension-tracing/issues/79), which we resolved in pull request [#86](https://github.com/AxonFramework/extension-tracing/pull/86).

You can find a list of all changes made in release 4.4 [here](https://github.com/AxonFramework/extension-tracing/issues?q=is%3Aclosed+milestone%3A%22Release+4.4%22).

## Release 4.3

* Pull request [#33](https://github.com/AxonFramework/extension-tracing/pull/33) introduces tracing for scatter-gather and subscription queries.
  This introduction means that we can make a full release of the Axon Tracing Extension.

* Traces for query messages now have tags included.
  We also introduced additional test cases to validate the entire process, as seen in [this](https://github.com/AxonFramework/extension-tracing/pull/42) issue.

For a complete list of all changes, we refer to [this](https://github.com/AxonFramework/extension-tracing/issues?q=is%3Aclosed+milestone%3A%22Release+4.3%22) page.

## Release 4.2 - Milestone

We did not introduce any significant changes other than updating the extension to use Axon Framework release 4.2.

Note that this extension currently is in a _milestone_ state.
As such, users should consider we might introduce API changes in future releases.

## Release 4.1 - Milestone

* The constructors of the `TracingCommandGateway` and `TracingQueryGateway` are now protected, since issue [#9](https://github.com/AxonFramework/extension-tracing/issues/9).
  This change allows users to extend these classes if necessary.

* As off issue [#7](https://github.com/AxonFramework/extension-tracing/issues/7), the spans now contain defaults operation names and tags.

You can find a complete list of the changes [here](https://github.com/AxonFramework/extension-tracing/issues?q=is%3Aclosed+milestone%3A%22Release+4.1%22)

Note that this extension currently is in a _milestone_ state.
As such, users should consider we might introduce API changes in future releases.

## Release 4.0 - Milestone

We introduced the Tracing extension with lots of help from our contributor Christophe Bouhier at Trifork.
The tracing logic used originates from the [Open Tracing API](https://opentracing.io/).

You can find a complete list of all changes [here](https://github.com/AxonFramework/extension-tracing/issues?q=is%3Aclosed+milestone%3A%22Release+4.0%22).

It's currently in a _milestone_ state, as it doesn't trace all `QueryGateway` operations.
As such, users should consider we might introduce API changes in future releases.
