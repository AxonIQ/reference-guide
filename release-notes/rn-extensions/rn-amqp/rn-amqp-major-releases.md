# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon AMQP Extension.

## Release 4.7.0

### Enhancements

- Add support for Spring boot 3. [#320](https://github.com/AxonFramework/extension-amqp/pull/320)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@gklijs](https://github.com/gklijs)

## Release 4.6.0

If you're curious about the dependency upgrades made in this release we refer to [this](https://github.com/AxonFramework/extension-amqp/releases/tag/axon-amqp-4.6.0) page.

### Enhancements

- Change how Sonar is invoked for GHAs [#162](https://github.com/AxonFramework/extension-amqp/pull/162)
- Added an AMQP sample to show how to use the extension [#126](https://github.com/AxonFramework/extension-amqp/pull/126)
- Splitted builds into pr and not pr, added ghactions to dependabot and other minors [#88](https://github.com/AxonFramework/extension-amqp/pull/88)

### Bug Fixes

- Precision on Date Time Utils [#94](https://github.com/AxonFramework/extension-amqp/issues/94)

### Contributors

We'd like to thank all the contributors who worked on this release!

- [@smcvb](https://github.com/smcvb)
- [@lfgcampos](https://github.com/lfgcampos)

## Release 4.5

* Issue [#5](https://github.com/AxonFramework/extension-amqp/pull/5) introduces dependabot.
  This updated a large amount of the versions used by this extension.

* We updated all tests to a more current testing solution.
  This means we introduced [Testcontainers](https://www.testcontainers.org/) and updated JUnit to version 5.
  You can find the made changes in [this](https://github.com/AxonFramework/extension-amqp/pull/44) pull request.

You can check [this](https://github.com/AxonFramework/extension-amqp/releases/tag/axon-amqp-4.5) page for a complete list of all changes.

## Release 4.4

* Issue [#3](https://github.com/AxonFramework/extension-amqp/pull/3) introduces support for Spring Boot Developer Tools.

* We introduce dependabot, which updated the log4j version. 

## Release 4.3

We did not introduce any significant changes other than updating the extension to use Axon Framework release 4.3.

## Release 4.2

We did not introduce any significant changes other than updating the extension to use Axon Framework release 4.2.

## Release 4.1

A version predicament caused the extension to load the wrong package for the `ChannelAwareMessageListener`.
We resolved this issue for release 4.1 [here](https://github.com/AxonFramework/extension-amqp/issues/1).

Furthermore, we updated the dependency on Axon Framework to 4.1.

## Release 4.0

We split off the AMQP logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
