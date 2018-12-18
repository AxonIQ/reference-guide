# Architecture overview

-DRAFT version 1.0.0-

 - Focus on message-driven style first (commands, events and queries)
 - CQRS
 - location transparency
 - replace the arch image with the latest


**C**ommand **Q**uery **R**esponsibility **S**egregation \(CQRS\) on itself is a very simple pattern. It only prescribes that the component of an application that processes commands should be separated from the component that processes queries. Although this separation is very simple on itself, it provides a number of very powerful features when combined with other patterns. Axon provides the building blocks that make it easier to implement the different patterns that can be used in combination with CQRS.

The diagram below shows an example of an extended layout of a CQRS-based event driven architecture. The UI component, displayed on the left, interacts with the rest of the application in two ways: it sends commands to the application \(shown in the top section\), and it queries the application for information \(shown in the bottom section\).

![Architecture overview of a CQRS application](/.gitbook/assets/detailed-architecture-overview.png)

Commands are typically represented by simple and straightforward objects that contain all data necessary for a command handler to execute it. A command expresses its intent by its name. In Java terms, that means the class name is used to figure out what needs to be done, and the fields of the command provide the information required to do it.

The Command Bus receives commands and routes them to the command handlers. Each command handler responds to a specific type of command and executes logic based on the contents of the command. In some cases, however, you would also want to execute logic regardless of the actual type of command, such as validation, logging or authorization.

The command handler retrieves domain objects \(Aggregates\) from a repository and executes methods on them to change their state. These aggregates typically contain the actual business logic and are therefore responsible for guarding their own invariants. The state changes of aggregates result in the generation of Domain Events. Both the Domain Events and the Aggregates form the domain model.

Repositories are responsible for providing access to aggregates. Typically, these repositories are optimized for lookup of an aggregate by its unique identifier only. Some repositories will store the state of the aggregate itself \(using Object Relational Mapping, for example\), while others store the state changes that the aggregate has gone through in an event store. The repository is also responsible for persisting the changes made to aggregates in its backing storage.

Axon provides support for both the direct way of persisting aggregates \(using Object Relational Mapping, for example\) and for event sourcing.

The event bus dispatches events to all interested event listeners. This can either be done synchronously or asynchronously. Asynchronous event dispatching allows the command execution to return and hand over control to the user, while the events are being dispatched and processed in the background. Not having to wait for event processing to complete makes an application more responsive. Synchronous event processing, on the other hand, is simpler and is a sensible default. By default, synchronous processing will process event listeners in the same transaction that also processed the command.

Event listeners receive events and handle them. Some handlers will update data sources used for querying while others send messages to external systems. As you might notice, the command handlers are completely unaware of the components that are interested in the changes they make. This means that it is very non-intrusive to extend the application with new functionality. All you need to do is add another event listener. The events loosely couple all components in your application together.

In some cases, event processing requires new commands to be sent to the application. An example of this is when an order is received. This could mean the customer's account should be debited with the amount of the purchase, and shipping must be told to prepare a shipment of the purchased goods. In many applications, logic will become more complicated than this: what if the customer didn't pay in time? Will you send the shipment right away, or await payment first? The saga is the CQRS concept responsible for managing these complex business transactions.

The framework provides components to handle queries. The query bus receives queries and routes them to the query handlers. A query handler is registered at the query bus with both the type of query it handles as well as the type of response it providers. Both the query and the result type are typically simple, read-only DTO objects. The contents of these DTOs are typically driven by the needs of the user interface. In most cases, they map directly to a specific view in the UI \(also referred to as table-per-view\).

It is possible to register multiple query handlers for the same type of query and type of response. When dispatching queries, the client can indicate whether he wants a result from one or from all available query handlers.
