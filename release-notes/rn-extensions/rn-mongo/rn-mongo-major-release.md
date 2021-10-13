# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Mongo Extension.

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
