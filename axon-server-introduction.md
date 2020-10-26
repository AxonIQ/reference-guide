# Axon Server

In a message-driven microservices environment it is extremely important that communication between services is efficient, reliable and easy to manage and monitor. Routing of messages should not require any manual configuration and adding new services should be easy to do.

With Axon Framework the internals of communication are hidden for the application developers. Axon Server provides this same experience in a distributed environment. It is an easy-to-use, easy-to-manage platform to handle all events, commands and queries.

Axon Server has knowledge about the different types of messages that are being exchanged: events, sent from one service to one or many other services, to notify the services that something has happened, commands, sent to one service to do something, potentially waiting for a result queries, sent to one or more services to retrieve information.

Each of these messages requires a different strategy, all supported by Axon Server.

Applications connect to the messaging platform and register their capabilities. One application may be able to execute a specific set of commands, another may handle a number of queries. There may be multiple instances of the same application connected to the messaging platform, each instance having the same capabilities.

## Message patterns

A client or application sends a request to the messaging platform. The platform finds the appropriate connected application\(s\) to send the request to, and forwards the request. It receives the reply, or replies, and forwards that to the caller. Commands are always sent to exactly one application. Commands for the same aggregate are always sent to the same application instance, to avoid problems with concurrent updates of the aggregate. Queries are sent to all applications capable of answering the query. If there are multiple instances of the same application, the query is only sent to one of the instances. Events are stored in the event store and sent to all registered listeners.

## High Available

Axon Server can operate in clustered mode. Each node in the cluster is active and applications are dynamically load balanced over the nodes. Instances of the same application will connect to the same messaging node to optimize performance. Instances of different applications are distributed over the nodes to spread the load.

## Flow control

Axon Server controls the flow of messages sent to the message handlers. The message handlers sent a number of permits to the messaging platform, indicating the number of messages the messaging platform may send. Once the handler is ready for more request it sends another message with a number of additional permits. Axon Server queues messages when there are no permits for the handler left. When a handler is disconnected while there are still queued messages, these are re-routed to another handler \(if possible\).

## QoS Messages

Clients can indicate priority of their request. This way, for example, commands originating from a batch process can be executed with lower priority than online request.

## Access control

Applications need to be granted access to the messaging platform. This avoids the risk that random clients can start sending commands into the system.

## Implementation technology

Axon Server has been developed fully in Java, building on Spring Boot. It is distributed as a single jar file containing all libraries used through shading.

Configuration information for Axon Server is stored in a small h2 database. This contains the information about the messaging platform nodes and the applications that have access. This information is automatically replicated between nodes in the cluster.

The messaging platform has 2 types of interfaces:

* HTTP
* gRPC \(HTTP 2.0\)

Communication between the standard Axon Framework Axon Server client and the messaging platform uses gRPC.

