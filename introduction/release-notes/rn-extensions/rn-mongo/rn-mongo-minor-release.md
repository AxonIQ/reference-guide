# Minor Releases

Any patch release made for an Axon project is tailored towards resolving bugs. This page aims to provide a dedicated overview of patch releases per project.

## _Release 4.0_

### Release 4.0.1

The Mongo extension incorrectly used the content type instead of the tokens type upon storing a serialized token.
The issue was marked and resolved under [#1](https://github.com/AxonFramework/extension-mongo/issues/1).

### Release 4.1.1

Axon Framework release 4.1 saw the introduction of split and merge operations.
These impact the `TokenStore` interface, more specifically with the `initializeSegment(TrackingToken, String, int) and `deleteToken(String, int)`.

Issue [#3](https://github.com/AxonFramework/extension-mongo/issues/3) marks the requirement for the `MongoTokenStore` to implement these as well.
