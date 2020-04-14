# Table of contents

* [Introduction](README.md)
* [Quick Start](quick-start.md)
* [Architecture Overview](architecture-overview/README.md)
  * [DDD & CQRS Concepts](architecture-overview/ddd-cqrs-concepts.md)
  * [Event-Driven Microservices](architecture-overview/event-driven-microservices.md)
* [Axon Server](axon-server-introduction.md)
* [Release Notes](release-notes/README.md)
  * [Bug Fixes](release-notes/bug-fixes.md)

## Getting Started

* [Prerequisites](getting-started/prerequisites.md)
* [Maven/Gradle Dependencies](getting-started/maven-dependencies.md)
* [Spring Boot](getting-started/spring-boot.md)
* [Starting the Axon Server](getting-started/starting-the-axon-server.md)

## Axon Framework

* [Introduction](axon-framework/introduction.md)
* [Messaging Concepts](axon-framework/messaging-concepts/README.md)
  * [Anatomy of a Message](axon-framework/messaging-concepts/anatomy-message.md)
  * [Message Intercepting](axon-framework/messaging-concepts/message-intercepting.md)
  * [Supported Parameters for Annotated Handlers](axon-framework/messaging-concepts/supported-parameters-annotated-handlers.md)
  * [Unit of Work](axon-framework/messaging-concepts/unit-of-work.md)
* [Commands](axon-framework/axon-framework-commands/README.md)
  * [Modeling](axon-framework/axon-framework-commands/modeling/README.md)
    * [Aggregate](axon-framework/axon-framework-commands/modeling/aggregate.md)
    * [Multi-Entity Aggregates](axon-framework/axon-framework-commands/modeling/multi-entity-aggregates.md)
    * [State Stored Aggregates](axon-framework/axon-framework-commands/modeling/state-stored-aggregates.md)
    * [Aggregate Creation from another Aggregate](axon-framework/axon-framework-commands/modeling/aggregate-creation-from-another-aggregate.md)
    * [Aggregate Polymorphism](axon-framework/axon-framework-commands/modeling/aggregate-polymorphism.md)
    * [Conflict Resolution](axon-framework/axon-framework-commands/modeling/conflict-resolution.md)
  * [Command Dispatchers](axon-framework/axon-framework-commands/command-dispatchers.md)
  * [Command Handlers](axon-framework/axon-framework-commands/command-handlers.md)
  * [Implementations](axon-framework/axon-framework-commands/implementations.md)
  * [Configuration](axon-framework/axon-framework-commands/configuration.md)
  * [Exception Handling](axon-framework/axon-framework-commands/exception-handling.md)
* [Events](axon-framework/events.md)
* [Queries](axon-framework/queries.md)
* [Sagas](axon-framework/sagas.md)
* [Deadlines](axon-framework/deadlines.md)
* [Testing](axon-framework/testing.md)
* [Tuning](axon-framework/tuning.md)
* [Monitoring and Metrics](axon-framework/monitoring-and-metrics.md)

## Axon Server

* [Introduction](axon-server/introduction.md)
* [Installation](axon-server/installation.md)
* [Administration](axon-server/administration.md)
* [Security](axon-server/security.md)
* [Performance](axon-server/performance.md)
* [Configuration](axon-server/configuration.md)

## Extensions

* [Spring AMQP](extensions/spring-amqp.md)
* [Kafka](extensions/kafka.md)
* [JGroups](extensions/jgroups.md)
* [Spring Cloud](extensions/spring-cloud.md)
* [Mongo](extensions/mongo.md)
* [Tracing](extensions/tracing.md)

## Appendices

* [A. RDBMS Tuning](appendices/rdbms-tuning.md)
* [B. Message Handler Tuning](appendices/message-handler-tuning/README.md)
  * [Parameter Resolvers](appendices/message-handler-tuning/parameter-resolvers.md)
  * [Handler Enhancers](appendices/message-handler-tuning/handler-enhancers.md)
* [C. Meta Annotations](appendices/meta-annotations.md)
* [D. Identifier Generation](appendices/identifier-generation.md)
* [E. Axon Server Query Language](appendices/query-reference.md)

