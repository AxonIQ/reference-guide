# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Mongo Extension.

## Release 4.1

Contributor `mwlynch` introduced a small improvement on the `MongoTokenStore` and `MongoEventStorageEngine`.
Namely, he removed the `@PostConstruct` annotation from the `ensureIndexes()` methods and moved the invocation to the constructor.
For the specific changes, we refer to [this](https://github.com/AxonFramework/extension-mongo/pull/2) pull request.

## Release 4.0

We split off the Mongo logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
