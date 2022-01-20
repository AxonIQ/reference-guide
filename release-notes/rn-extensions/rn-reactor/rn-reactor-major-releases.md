# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Reactor Extension.

## Release 4.5

* We introduce dependabot to this project in issue [#10](https://github.com/AxonFramework/extension-reactor/pull/10).

* We added automatic release note construction through Git Workflow in issue [#25](https://github.com/AxonFramework/extension-reactor/pull/25). 

All changes made can be found in the [release notes](https://github.com/AxonFramework/extension-reactor/releases/tag/axon-reactor-4.5).

## Release 4.4

We combined release 4.4.2 of Axon Framework with the introduction of the new Reactor Extension.
This extension provides gateway implementations that utilize `Mono` and `Flux` from [Project Reactor](https://projectreactor.io/).

We introduced this extension to support a more reactive programming style when using Axon's outer layer through the `CommandGatewy`, `EventGateway` and `QueryGateway`.