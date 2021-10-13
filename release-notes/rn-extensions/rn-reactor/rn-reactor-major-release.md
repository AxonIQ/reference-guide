# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Reactor Extension.

## Release 4.4

We combined release 4.4.2 of Axon Framework with the introduction of the new Reactor Extension.
This extension provides gateway implementations that utilize `Mono` and `Flux` from [Project Reactor](https://projectreactor.io/).

We introduced this extension to support a more reactive programming style when using Axon's outer layer through the `CommandGatewy`, `EventGateway` and `QueryGateway`.