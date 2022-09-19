# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Mongo Extension.

## Release 4.6

If you're curious about the dependency upgrades made in this release we refer to [this](https://github.com/AxonFramework/extension-mongo/releases/tag/axon-mongo-4.6.0) page.

### Enhancements

- Resolving some sonar issues and adding some concurrent tests. [#222](https://github.com/AxonFramework/extension-mongo/pull/222)
- Change how Sonar is invoked for GHA's [#159](https://github.com/AxonFramework/extension-mongo/pull/159)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@gklijs](https://github.com/gklijs)
- [@smcvb](https://github.com/smcvb)

## Release 4.5

* We introduced a dedicated sample project to the Mongo Extension.
  The sample is written in Kotlin and shows how to use the `MongoEventStorageEngine` and `MongoTokenStore`.
  For those interested, you can find the pull request [here](https://github.com/AxonFramework/extension-mongo/pull/65).

* Contributor `erikrz` provided an issue **and** pull request to allow the `MongoTokenStore` index to be automatically validated.
  Issue [#51](https://github.com/AxonFramework/extension-mongo/issues/51) marks the request, whereas [#53](https://github.com/AxonFramework/extension-mongo/pull/53) marks the pull request.

* The `MongoTokenStore` now implements the `retrieveStorageIdentifier` method, as of pull request [#139](https://github.com/AxonFramework/extension-mongo/pull/139).
  This change allows clustered environments, like Axon Server, to tell apart the Mongo Databases of Event Processors. 
  This further benefits split, merge and release operations performed in a distributed environment.

We refer to the [release notes](https://github.com/AxonFramework/extension-mongo/releases/tag/axon-mongo-4.5) for a complete list of all changes.

## Release 4.4

* Issue [#10](https://github.com/AxonFramework/extension-mongo/pull/10) introduces support for Spring Boot Developer Tools.

* Contributor `bjornharvold` provided a pull request to update the MongoDB java driver to 4.0.3.
  For those interested in the adjustments, you can check them out [here](https://github.com/AxonFramework/extension-mongo/pull/12).

We refer to [this](https://github.com/AxonFramework/extension-mongo/issues?q=is%3Aclosed+milestone%3A%22Release+4.4%22) page for a list of all changes.

## Release 4.3

We did not introduce any significant changes other than updating the extension to use Axon Framework release 4.3.

## Release 4.2

We did not introduce any significant changes other than updating the extension to use Axon Framework release 4.2.

## Release 4.1

Contributor `mwlynch` introduced a small improvement on the `MongoTokenStore` and `MongoEventStorageEngine`.
Namely, he removed the `@PostConstruct` annotation from the `ensureIndexes()` methods and moved the invocation to the constructor.
For the specific changes, we refer to [this](https://github.com/AxonFramework/extension-mongo/pull/2) pull request.

## Release 4.0

We split off the Mongo logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
