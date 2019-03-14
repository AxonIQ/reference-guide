# Table of contents

* [Quick start](introduction/quick-start.md)
* [Architecture overview](introduction/architecture-overview/architecture-overview.md)
    * [DDD & CQRS concepts](introduction/architecture-overview/ddd-cqrs-concepts.md)
    * [Event-Driven Microservices](introduction/architecture-overview/event-driven-microservices.md)
* [Axon Server](introduction/axon-server.md)

## Setting up

* [Prerequisites](setting-up/prerequisites.md)
* [Maven/Gradle dependencies](setting-up/maven-dependencies.md)
* [Spring Boot](setting-up/spring-boot.md)
* [Starting the Axon Server](setting-up/starting-the-axon-server.md)

## Implementing domain logic

* [Command handling](implementing-domain-logic/command-handling/command-handling.md)
    * [Aggregate](implementing-domain-logic/command-handling/aggregate.md)
    * [Multi-entity Aggregates](implementing-domain-logic/command-handling/multi-entity-aggregates.md)
    * [External Command Handlers](implementing-domain-logic/command-handling/external-command-handler.md)
    * [State-Stored Aggregates](implementing-domain-logic/command-handling/state-stored-aggregates.md)
    * [Dispatching Commands](implementing-domain-logic/command-handling/dispatching-commands.md)
    * [Testing](implementing-domain-logic/command-handling/testing.md)
    * [Aggregate creation from another Aggregate](implementing-domain-logic/command-handling/aggregate-creation-from-aggregate.md)
    * [Conflict resolution](implementing-domain-logic/command-handling/conflict-resolution.md)
* [Event handling](implementing-domain-logic/event-handling/event-handling.md)
    * [Handling Events](implementing-domain-logic/event-handling/handling-events.md)
    * [Dispatching Events](implementing-domain-logic/event-handling/dispatching-events.md)
* [Query handling](implementing-domain-logic/query-handling/query-handling.md)
    * [Handling Queries](implementing-domain-logic/query-handling/handling-queries.md)
    * [Dispatching Queries](implementing-domain-logic/query-handling/dispatching-queries.md)
* [Complex Business Transactions](implementing-domain-logic/complex-business-transactions/complex-business-transactions.md)
    * [Implementing a Saga](implementing-domain-logic/complex-business-transactions/implementing-saga.md)
    * [Deadline handling](implementing-domain-logic/complex-business-transactions/deadline-handling.md)
    * [Managing Associations](implementing-domain-logic/complex-business-transactions/managing-associations.md)
    * [Testing a Saga](implementing-domain-logic/complex-business-transactions/testing.md)

## Configuring Infrastructure components

* [Messaging concepts](configuring-infrastructure-components/messaging-concepts/messaging-concepts.md)
    * [Anatomy of a Message](configuring-infrastructure-components/messaging-concepts/message-anatomy.md)
    * [Message intercepting](configuring-infrastructure-components/messaging-concepts/message-intercepting.md)
    * [Supported Parameters for annotated handlers](configuring-infrastructure-components/messaging-concepts/supported-parameters-for-annotated-handlers.md)
    * [Unit of Work](configuring-infrastructure-components/messaging-concepts/unit-of-work.md)
* [Command processing](configuring-infrastructure-components/command-processing/command-processing.md)
    * [Command Model Configuration](configuring-infrastructure-components/command-processing/command-model-configuration.md)
    * [Command dispatching](configuring-infrastructure-components/command-processing/command-dispatching.md)
    * [Optimizing Aggregate loading](configuring-infrastructure-components/command-processing/optimizing-aggregate-loading.md)
* [Event processing](configuring-infrastructure-components/event-processing/event-processing.md)
    * [Event Bus & Event Store](configuring-infrastructure-components/event-processing/event-bus-and-event-store.md)
    * [Saga infrastructure](configuring-infrastructure-components/event-processing/saga-infrastructure.md)
    * [Event Processors](configuring-infrastructure-components/event-processing/event-processors.md)
* [Query processing](configuring-infrastructure-components/query-processing.md)
* [Deadlines](configuring-infrastructure-components/deadlines.md)

## Operations Guide 

* [Setting up Axon Server](operations-guide/setting-up-axon-server/setting-up-axon-server.md)
    * [Launch](operations-guide/setting-up-axon-server/launch.md)
    * [Tuning](operations-guide/setting-up-axon-server/tuning.md)
    * [Backups](operations-guide/setting-up-axon-server/backups.md)
    * [Monitoring](operations-guide/setting-up-axon-server/monitoring.md)
    * [Access Control](operations-guide/setting-up-axon-server/access-control.md)
    * [Axon Server clustering](operations-guide/setting-up-axon-server/axon-server-clustering.md)
    * [Multi-context](operations-guide/setting-up-axon-server/multi-context.md)
    * [Command-line interface](operations-guide/setting-up-axon-server/command-line.md)
* [Production considerations](operations-guide/production-considerations/production-considerations.md)
    * [Versioning Events](operations-guide/production-considerations/versioning-events.md)
    * [Serializers](operations-guide/production-considerations/serializers.md)
    * [Monitoring and Metrics](operations-guide/production-considerations/monitoring-and-metrics.md)

## Extensions

* [Spring AMQP](extensions/spring-amqp.md)
* [Kafka](extensions/kafka.md)
* [JGroups](extensions/jgroups.md)
* [Spring Cloud](extensions/spring-cloud.md)
* [Mongo](extensions/mongo.md)

## Appendices

* [A. RDBMS Tuning](appendices/rdbms-tuning.md)
* [B. Message Handler Tuning](appendices/message-handler-tuning/message-handler-tuning.md)
    * [Parameter Resolvers](appendices/message-handler-tuning/parameter-resolvers.md)
    * [Handler Enhancers](appendices/message-handler-tuning/handler-enhancers.md)
* [C. Meta Annotations](appendices/meta-annotations.md)
* [D. Identifier Generation](appendices/identifier-generation.md)
