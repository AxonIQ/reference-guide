# Event-Driven Microservices

The concepts described in the [DDD and CQRS Concepts](ddd-cqrs-concepts.md) chapter are highly applicable when designing and creating \(event-driven\) microservices systems. In this chapter, we will explicitly name a few common strategies for applying Axon in such environments.

## Evolutionary microservices

At AxonIQ, we believe systems evolve their way into microservices, instead of attempting to build a microservices system from scratch. The main reason is that the exploration of sensible Context Boundaries \(see [Bounded Context](ddd-cqrs-concepts.md#bounded-context)\) and models takes time. Changing these boundaries is much more difficult in distributed systems than it is in a monolith.

Axon leverages the separation of components and uses explicit messaging between them, which makes these components [Location Transparent](./#location-transparency). Unlike the use of service discovery, the approach Axon takes for messaging doesn't require a component to know the destination of a message at all. They are automatically routed to a component that advertises the capability to handle such messages. This makes these systems much more flexible to change than "regular" microservices based systems.

## Strategies for applying Axon

There are different strategies for applying Axon in Microservices environments. One could adopt the Axon philosophy on a system level and build all services using Axon. However, Axon is also already useful when just applying it within a single application/service. Lastly, we'll also discuss specific strategies when using Axon in a polyglot environment. To that end, Axon is built with integration in mind.

### Axon based microservices

When using the Axon on a system level, meaning that several services run Axon \(or compatible APIs\), one can use the messaging concepts to their fullest extent. Applications can simply make use of the different message buses to send and receive messages from other components. This makes the system very flexible when it comes to changing the deployment strategy of components.

### Axon within a single service only

When building a single Axon based service within an existing Microservices system, you may want to expose your API using "traditional" rest endpoints. In such case, your Axon based application would need a small API layer that transforms REST calls into commands, which are then dispatched to a command bus internally. Do take into account, however, that requests may not be routed consistently, and that commands for the same aggregate may then be routed to different instances.

If an Axon based service has several instances deployed, you may still benefit from using distributed implementations of the buses, to allow these instances to properly balance message handling between them.

### Hybrid / polyglot environments

In practice, many microservices based systems run in a polyglot environment. Different services will run on a different technology stack. In these environments, it is even more important to ensure Contexts Boundaries are properly guarded and provide decent anti-corruption layers, where applicable.

It is unlikely that all used technology stacks follow the same messaging based approach that Axon applications do. However, that doesn't mean the concepts need to be abandoned. You can still benefit from many of the advantages of explicit messaging. In such environments, anti-corruption layers could be implemented as components that handle commands, events and queries, and execute other types of calls \(e.g. REST calls\) to the external services. This way, components that work with explicit messaging do not need to worry about polling external services for changes, or be influenced by the technical challenges caused by different types of APIs.

Axon supports different types of connectors that allow Events \(and in certain cases other message types, too\) to be published to third party message brokers. By default, Axon will make assumptions on the format of these external events, but they can always be overridden. Read the chapters about these specific extensions for more details.

