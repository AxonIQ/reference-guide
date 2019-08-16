# Bug Fixes

Any patch release made for an Axon project is tailored towards resolving bugs.
This page aims to provide a dedicated overview of patch releases per project.   

## Axon Framework

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

### Axon Mongo Extension 4.0.1

The Mongo extension incorrectly used the content type instead of the tokens type upon storing a serialized token.
The issue was marked and resolved under [#1](https://github.com/AxonFramework/extension-mongo/issues/1).