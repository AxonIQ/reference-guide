# Bug Fixes

Any patch release made for an Axon project is tailored towards resolving bugs.
This page aims to provide a dedicated overview of patch releases per project.   

## Axon Framework

### Release 4.1.2

 * A dependency on `XStream` was enforced undesirably through the Builder pattern introduced in 4.0. 
   This has been resolved by using a `Supplier` of a `Serializer` in the Builders instead, as described under [this](https://github.com/AxonFramework/AxonFramework/issues/1054) issue.
 * Due to a hierarchy issue in the Spring Boot auto configuration, the `JdbcTokenStore` was not always used as expected.
   The ordering has been fixed under issue [#1077](https://github.com/AxonFramework/AxonFramework/issues/1077).
 * The ordering of message handling functions was incorrect according to the documentation.
   Classes take precedence over interface, and the depth of interface hierarchy is calculated based on the inheritance level (as described [here](https://github.com/AxonFramework/AxonFramework/pull/1129)).
 
For a complete list of all resolved bugs we refer to the
 [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.1.1

 * Query Dispatch Interceptors were no called correctly when a [subscription query](../../implementing-domain-logic/query-handling/dispatching-queries.md#subscription-queries) was performed when Axon Server was used as the `QueryBus`.
   This issue was marked [here](https://github.com/AxonFramework/AxonFramework/issues/1013) and resolved in pull request [#1042](https://github.com/AxonFramework/AxonFramework/pull/1042).
 * When Axon Server was (auto) configured without being able to connect to an actual instance, processing instructions were incorrectly dispatched regardless.
   Pull request [#1040](https://github.com/AxonFramework/AxonFramework/pull/1040) resolves this by making sure an active connection is present.
 * The Spring Boot auto configuration did not allow the exclusion of the `axon-server-connector` dependency due to a direct dependency on classes.
   This has been resolved by expecting fully qualified class names as Strings instead (resolved under [this](https://github.com/AxonFramework/AxonFramework/pull/1041) pull request).
 * The `JpaEventStorageEngine` was not wrapping the `appendEvents` operation in a transaction.
   Problem has been resolved under issue [#1035](https://github.com/AxonFramework/AxonFramework/issues/1035). 
 
For a complete list of all resolved bugs we refer to the
 [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1.1%22++label%3A%22Type%3A+Bug%22). 

### Release 4.1

 * Command an Query message priority was not set correctly for the Axon Server Connector.
   This issue has been addressed under bug [#1004](https://github.com/AxonFramework/AxonFramework/pull/1004).
 * The `CapacityMonitor` was not registered correctly for Event Processor, which user "Sabartius" resolved under issue [#961](https://github.com/AxonFramework/AxonFramework/issues/961).
 * Some exception were not reported correctly and/or clearly when utilizing the Axon Server Connector (issue marked under number [#945](https://github.com/AxonFramework/AxonFramework/pull/945)).
 
For a complete list of all resolved bugs we refer to the
 [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.1%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.4

 * Deserialization failures were accidentally swallowed by the command and query gateway (marked under [#967](https://github.com/AxonFramework/AxonFramework/issues/967)).
 * Resolved an issue where custom exception in a Command Handling constructor caused `NullPointerExceptions`.
 
For a complete list of all resolved bugs we refer to the
 [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.4%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.3

 * The `SimpleQueryBus` reported exceptions on the initial result incorrectly upon performing a subscription query.
   Issue has been described and resolved under [#913](https://github.com/AxonFramework/AxonFramework/issues/913). 
 * Resolved issue where the the "download Axon Server" message was shown upon a reconnect of an application to a Axon Server node.
 * Large global index gaps between events caused issues when querying the event stream (described [here](https://github.com/AxonFramework/AxonFramework/issues/419)).
 * Fixed inconsistency in the `GlobalSequenceTrackingToken#covers(TrackingToken)` method.   
 
For a complete list of all resolved bugs we refer to the
 [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.3%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.2

 * A timeout was thrown instead of a exception by Axon Server when a duplicate aggregate id was created, which is resolved in [#903](https://github.com/AxonFramework/AxonFramework/issues/903). 
 * Command or Query handling exception were no properly serialized through Axon Server (resolved in [#904](https://github.com/AxonFramework/AxonFramework/pull/904)). 
 
For a complete list of all resolved bugs we refer to the
 [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.2%22++label%3A%22Type%3A+Bug%22).

### Release 4.0.1

 * Resolved `QueryUpdateEmitter` configuration for the Axon Server connector set up (see issue [here](https://github.com/AxonFramework/AxonFramework/issues/896)).
 * For migration purposes legacy `TrackingTokens` should have been added, which is resolved [here](https://github.com/AxonFramework/AxonFramework/issues/886).
 * Event Processing was stopped after a reconnection with Axon Server. Resolve the problem in issue [#883](https://github.com/AxonFramework/AxonFramework/issues/883). 
 
For a complete list of all resolved bugs we refer to the
 [issue tracker](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0.1%22++label%3A%22Type%3A+Bug%22).

### Release 4.0

The bugs marked for release 4.0 were issues introduced to new features or enhancements.
As such they should not have impacted users in any way.
Regardless, the full list can be found
 [here](https://github.com/AxonFramework/AxonFramework/issues?utf8=%E2%9C%93&q=is%3Aclosed+milestone%3A%22Release+4.0%22++label%3A%22Type%3A+Bug%22).

## Axon Framework Extensions

### Axon JGroups Extension 4.1

If cluster connection message came in quickly after starting the connection, a `NullPointerException` could be thrown.
This issue was resolved for release 4.1 [here](https://github.com/AxonFramework/extension-jgroups/issues/1).

### Axon Mongo Extension 4.0.1

The Mongo extension incorrectly used the content type instead of the tokens type upon storing a serialized token.
The issue was marked and resolved under [#1](https://github.com/AxonFramework/extension-mongo/issues/1).

### Axon Spring Cloud Extension 4.1

The `SpringCloudCommandRouter` failed to correctly connect to a Spring Cloud Discovery Service if the node did not contain any Command Handler methods.
This undesired behaviour was marked by user "travikk" and made more lenient under [this](https://github.com/AxonFramework/extension-springcloud/issues/1).
