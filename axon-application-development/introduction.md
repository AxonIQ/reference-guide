# Introduction

The Axon Framework provides the building blocks for applications that are based on architectural principles like [CQRS/Domain-Driven Design](../architecture-overview/ddd-cqrs-concepts.md) and [Event Sourcing](../architecture-overview/event-sourcing-tbd.md). The framework provides a proven set of technology concerns that enables developers to easily build their applications following these patterns without having to worry about the non-functional aspects required while adopting these patterns.

The Axon Framework is built on the following foundational principles

_Scalability_

Axon Framework enforces location transparency in its core APIs. Axon allows components to communicate through 3 different types of [Messages](messaging-concepts/): [Commands](command-handling/), [Queries](query-handling/) and [Events](event-handling/), each with distinct routing patterns. By enforcing this throughout all components in an application, Axon Framework ensures applications can be physically separated into microservices and scaled out at all times, without any changes to business logic.

_Performance_

Axon Framework distinguishes between two types of components; query \(read\) and update \(write\) components. This means the information structure in these components can be optimized for the purpose at hand. As a result, query components have quick access to data that is already structured for its purpose. This in itself is a performance benefit, but also allows scaling for a specific component type.

_Auditing_  

Axon Framework provides support for [Event Sourcing,](../architecture-overview/event-sourcing-tbd.md) providing the most reliable form of auditing, out of the box. Besides auditing, Event Sourcing has also proven to provide for a valuable source of information for analytics. The Event Stream provides a reliable trail of everything that happened, based on which new insights can be calculated.

_Agility_

As software systems evolve during their lifetime, they grow more complex. It is essential that complex systems themselves are built on simple, loosely coupled, components. Axon Framework provides clear and simple APIs for components to communicate in a location transparent way. This ensures that components remain loosely coupled, and can be simply interchanged with a new version.

_Integration_

The __Axon Framework works with explicit messaging concepts which means that you can easily add components which provides integration with 3rd party systems without intruding the original application behavior. This holds for both in- and outbound 3rd party systems. This architecture makes integration with several external systems possible, without increasing complexity.

