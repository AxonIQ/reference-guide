# Experimental Releases

This page notes all enhancements and features that we have introduced to our experimental releases of the Axon Kotlin Extension.

## Release 4.6

If you're curious about the dependency upgrades made in this release we refer to [this](https://github.com/AxonFramework/extension-kotlin/releases/tag/axon-kotlin-4.6.0) page.

### Features

- [#17] Subscription queries extensions [#196](https://github.com/AxonFramework/extension-kotlin/pull/196)
- Subscription queries extensions [#17](https://github.com/AxonFramework/extension-kotlin/issues/17)

### Enhancements

- Change how Sonar is invoked for GHAs [#168](https://github.com/AxonFramework/extension-kotlin/pull/168)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@smcvb](https://github.com/smcvb)
- [@rsobies](https://github.com/rsobies)

## Release 0.2

* Issue [#16](https://github.com/AxonFramework/extension-kotlin/issues/16) marked the desire to introduce extension methods for the scatter-gather query.
  Contributor `hsenasilva` answered this call and introduced them in pull request [#76](https://github.com/AxonFramework/extension-kotlin/pull/76).

* We introduced a dedicated kotlin test module, as further described in [this](https://github.com/AxonFramework/extension-kotlin/issues/67) issue.
  This module includes extension methods for the `AggregateTestFixture` and `SagaTestFixture`.

* Contributor `zambrovksi` added extension functions for the `AggregateLifecycle`, to further enhance the Kotlin experience from within an aggregate.
  For those interested, you can find the adjustments in [this](https://github.com/AxonFramework/extension-kotlin/pull/72) pull request.

* We introduce dependabot to the project in issue [#43](https://github.com/AxonFramework/extension-kotlin/pull/43).
  This introduction means a lot of the changes are towards version updates to the project.

You can check out all changes made in release 0.2.0 [here](https://github.com/AxonFramework/extension-kotlin/releases/tag/axon-kotlin-0.2.0).

## Release 0.1

Release 0.1.0 marks the first release of Axon's Kotlin Extension, adding extension methods in Kotlin for Axon Framework 
Some noteworthy additions are:

* Extension methods on the `CommandGateway`
* Extension methods on the `QueryGateway`
* Extension methods for event upcasters
* Extension methods on the `QueryUpdateEmitter`

You can check the milestone page for a complete list of the changes [here](https://github.com/AxonFramework/extension-kotlin/issues?q=is%3Aclosed+milestone%3A%22Release+0.1.0%22).