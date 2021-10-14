# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon JGroups Extension.

## Release 4.5

* We added automatic release note construction through Git Workflow in issue [#32](https://github.com/AxonFramework/extension-jgroups/pull/32).

You can check the release note [here](https://github.com/AxonFramework/extension-jgroups/releases/tag/axon-jgroups-4.5) for a complete list of all changes.

## Release 4.4

* Issue [#9](https://github.com/AxonFramework/extension-jgroups/pull/9) introduces support for Spring Boot Developer Tools.

* We introduce dependabot, which updated the log4j version.

## Release 4.3

* Issue [#7](https://github.com/AxonFramework/extension-jgroups/pull/7) implements the `CommandBusConnector#localSegment`.
  Axon Framework introduces this method in release 4.3 (and issue [#874](https://github.com/AxonFramework/AxonFramework/issues/874)) to ensure the usage of the `DisruptorCommandBus`'s repository is followed when distributing the `CommandBus`.

* We introduced a graceful start-up and shutdown solution in Axon Framework release 4.3 for all infrastructure components.
  Issue [#8](https://github.com/AxonFramework/extension-jgroups/pull/8) ensures the `JGroupsConnector` complies with this style too.

## Release 4.2

As marked under [this](https://github.com/AxonFramework/extension-jgroups/issues/4) issue, the command callback will
not be called if the connection between JGroups peers dies whilst a command is in transit.
Credits go to "sgrimm-sg" for filing the issue and solving [it](https://github.com/AxonFramework/extension-jgroups/pull/5).

## Release 4.1

If cluster connection message came in quickly after starting the connection, a `NullPointerException` could be thrown.
This issue was resolved for release 4.1 [here](https://github.com/AxonFramework/extension-jgroups/issues/1).

## Release 4.0

We split off the JGroups logic from Axon Framework core into a dedicated repository.
Next to that, it complies with Axon Framework's 4.0 release.
