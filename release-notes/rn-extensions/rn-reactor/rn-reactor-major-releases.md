# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Reactor Extension.

## Release 4.9

Release 4.9.0 only consists out of dependency upgrades to tag along with recent developments.

## Release 4.8

Release 4.8.0 only consists out of dependency upgrades to tag along with recent developments.

## Release 4.7

### Enhancements

- Fix coverage report, including the integration test module. [#180](https://github.com/AxonFramework/extension-reactor/pull/180)
- Enable Spring Boot 3 auto config [#177](https://github.com/AxonFramework/extension-reactor/pull/177)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@gklijs](https://github.com/gklijs)

## Release 4.6

If you're curious about the dependency upgrades made in this release we refer to [this](https://github.com/AxonFramework/extension-reactor/releases/tag/axon-reactor-4.6.0) page.

### Features

- Streaming query support [#78](https://github.com/AxonFramework/extension-reactor/pull/78)
- Support reactive query handlers and Flux/Mono query results [#70](https://github.com/AxonFramework/extension-reactor/issues/70)
- Streaming query - (Flux as Query handler return type) [#3](https://github.com/AxonFramework/extension-reactor/issues/3)

### Enhancements

- Splitted builds into pr and not pr, added ghactions to dependabot and other minors [#56](https://github.com/AxonFramework/extension-reactor/pull/56)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@lfgcampos](https://github.com/lfgcampos)
- [@schananas](https://github.com/schananas)

## Release 4.5

* We introduce dependabot to this project in issue [#10](https://github.com/AxonFramework/extension-reactor/pull/10).

* We added automatic release note construction through Git Workflow in issue [#25](https://github.com/AxonFramework/extension-reactor/pull/25). 

All changes made can be found in the [release notes](https://github.com/AxonFramework/extension-reactor/releases/tag/axon-reactor-4.5).

## Release 4.4

We combined release 4.4.2 of Axon Framework with the introduction of the new Reactor Extension.
This extension provides gateway implementations that utilize `Mono` and `Flux` from [Project Reactor](https://projectreactor.io/).

We introduced this extension to support a more reactive programming style when using Axon's outer layer through the `CommandGatewy`, `EventGateway` and `QueryGateway`.