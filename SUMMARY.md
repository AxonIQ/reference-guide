# Table of contents

* [Introduction](README.md)
* [Quick Start](quick-start.md)
* [Architecture Overview](architecture-overview/README.md)
  * [DDD & CQRS Concepts](architecture-overview/ddd-cqrs-concepts.md)
  * [Event-Driven Microservices](architecture-overview/event-driven-microservices.md)
* [Axon Server](axon-server.md)
* [Release Notes](release-notes/README.md)
  * [Bug Fixes](release-notes/bug-fixes.md)

## Setting Up

* [Prerequisites](setting-up/prerequisites.md)
* [Maven/Gradle Dependencies](setting-up/maven-dependencies.md)
* [Spring Boot](setting-up/spring-boot.md)
* [Starting the Axon Server](setting-up/starting-the-axon-server.md)

## Application Development <a id="axon-application-development"></a>

* [Command handling](axon-application-development/command-handling/README.md)
  * [Aggregate](axon-application-development/command-handling/aggregate.md)
  * [Multi-entity Aggregates](axon-application-development/command-handling/multi-entity-aggregates.md)
  * [Aggregate Polymorphism](axon-application-development/command-handling/aggregate-polymorphism.md)
  * [External Command Handlers](axon-application-development/command-handling/external-command-handler.md)
  * [State-Stored Aggregates](axon-application-development/command-handling/state-stored-aggregates.md)
  * [Dispatching Commands](axon-application-development/command-handling/dispatching-commands.md)
  * [Testing](axon-application-development/command-handling/testing.md)
  * [Aggregate Creation from another Aggregate](axon-application-development/command-handling/aggregate-creation-from-aggregate.md)
  * [Conflict Resolution](axon-application-development/command-handling/conflict-resolution.md)
* [Event handling](axon-application-development/event-handling/README.md)
  * [Handling Events](axon-application-development/event-handling/handling-events.md)
  * [Dispatching Events](axon-application-development/event-handling/dispatching-events.md)
* [Query Handling](axon-application-development/query-handling/README.md)
  * [Handling Queries](axon-application-development/query-handling/handling-queries.md)
  * [Dispatching Queries](axon-application-development/query-handling/dispatching-queries.md)
* [Complex Business Transactions](axon-application-development/complex-business-transactions/README.md)
  * [Implementing a Saga](axon-application-development/complex-business-transactions/implementing-saga.md)
  * [Deadline Handling](axon-application-development/complex-business-transactions/deadline-handling.md)
  * [Managing Associations](axon-application-development/complex-business-transactions/managing-associations.md)
  * [Testing a Saga](axon-application-development/complex-business-transactions/testing.md)

## Configuring Infrastructure Components

* [Messaging concepts](configuring-infrastructure-components/messaging-concepts/README.md)
  * [Anatomy of a Message](configuring-infrastructure-components/messaging-concepts/message-anatomy.md)
  * [Message Intercepting](configuring-infrastructure-components/messaging-concepts/message-intercepting.md)
  * [Supported Parameters for Annotated Handlers](configuring-infrastructure-components/messaging-concepts/supported-parameters-for-annotated-handlers.md)
  * [Unit of Work](configuring-infrastructure-components/messaging-concepts/unit-of-work.md)
* [Command Processing](configuring-infrastructure-components/command-processing/README.md)
  * [Command Model Configuration](configuring-infrastructure-components/command-processing/command-model-configuration.md)
  * [Command Dispatching](configuring-infrastructure-components/command-processing/command-dispatching.md)
  * [Optimizing Aggregate Loading](configuring-infrastructure-components/command-processing/optimizing-aggregate-loading.md)
* [Event Processing](configuring-infrastructure-components/event-processing/README.md)
  * [Event Bus & Event Store](configuring-infrastructure-components/event-processing/event-bus-and-event-store.md)
  * [Saga Infrastructure](configuring-infrastructure-components/event-processing/saga-infrastructure.md)
  * [Event Processors](configuring-infrastructure-components/event-processing/event-processors.md)
* [Query Processing](configuring-infrastructure-components/query-processing/README.md)
  * [Configuring Query Handlers](configuring-infrastructure-components/query-processing/configuring-query-handlers.md)
  * [Query Dispatching](configuring-infrastructure-components/query-processing/query-dispatching.md)
* [Deadlines](configuring-infrastructure-components/deadlines.md)

## Operations Guide

* [Setting up Axon Server](operations-guide/setting-up-axon-server/README.md)
  * [Launch](operations-guide/setting-up-axon-server/launch.md)
  * [Tuning](operations-guide/setting-up-axon-server/tuning.md)
  * [Backups](operations-guide/setting-up-axon-server/backups.md)
  * [Monitoring](operations-guide/setting-up-axon-server/monitoring.md)
  * [Access Control](operations-guide/setting-up-axon-server/access-control.md)
  * [Axon Server Clustering](operations-guide/setting-up-axon-server/axon-server-clustering.md)
  * [Multi-context](operations-guide/setting-up-axon-server/multi-context.md)
  * [Backup and messaging-only nodes](operations-guide/setting-up-axon-server/node-roles.md)
  * [Command-line Interface](operations-guide/setting-up-axon-server/command-line.md)
  * [Tagging](operations-guide/setting-up-axon-server/tagging.md)
  * [Development Mode](operations-guide/setting-up-axon-server/development-mode.md)
  * [Heartbeat Monitoring](operations-guide/setting-up-axon-server/heartbeat-monitoring.md)
  * [Recovery](operations-guide/setting-up-axon-server/recovery.md)
* [Production Considerations](operations-guide/production-considerations/README.md)
  * [Versioning Events](operations-guide/production-considerations/versioning-events.md)
  * [Serializers](operations-guide/production-considerations/serializers.md)
  * [Monitoring and Metrics](operations-guide/production-considerations/monitoring-and-metrics.md)
  * [Exceptions](operations-guide/production-considerations/exceptions.md)
* [Runtime Tuning](operations-guide/runtime-tuning/README.md)
  * [Tuning Event Processing](operations-guide/runtime-tuning/tuning-event-processing.md)
  * [Tuning Command Processing](operations-guide/runtime-tuning/tuning-command-processing.md)

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

