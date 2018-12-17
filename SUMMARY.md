# Table of contents

* [Quick start](introduction/_quick-start.md)
* [Architecture overview](introduction/architecture-overview/architecture-overview.md)
    * [DDD & CQRS concepts](introduction/architecture-overview/ddd-cqrs-patters.md)
    * [Event-Driven Microservices](introduction/architecture-overview/event-driven-microservices.md)
* [Axon server](introduction/axon-server.md)

## Setting up

* [Prerequisites](setting-up/_prerequisites.md)
<!--* [Code structure considerations](setting-up/_code-structure.md)-->
* [Maven/Gradle dependencies](setting-up/maven-dependencies.md)
* [Spring boot](setting-up/spring-boot.md)
<!--* [Java CDI](setting-up/_java-cdi.md)-->
* [Starting the Axon server](setting-up/starting-the-axon-server.md)

## Implementing domain logic

* [Command handling](implementing-domain-logic/command-handling/command-handling.md)
    * [Aggregate](implementing-domain-logic/command-handling/aggregate.md)
    * [External command handler](implementing-domain-logic/command-handling/external-command-handler.md)
    * [Multi-entity aggregates](implementing-domain-logic/command-handling/multi-entity-aggregates.md)
    * [State-stored aggregates](implementing-domain-logic/command-handling/state-stored-aggregates.md)
    * [Dispatching commands](implementing-domain-logic/command-handling/dispatching-commands.md)
    * [Testing](implementing-domain-logic/command-handling/testing.md)
* [Event handling](implementing-domain-logic/event-handling/event-handling.md)
    * [Handling events](implementing-domain-logic/event-handling/handling-events.md)
    * [Dispatching events](implementing-domain-logic/event-handling/dispatching-events.md)
    * [Updating view model](implementing-domain-logic/event-handling/updating-view-model.md)
* [Query handling](implementing-domain-logic/query-handling/query-handling.md)
    * [Handling queries](implementing-domain-logic/query-handling/handling-queries.md)
    * [Dispatching queries](implementing-domain-logic/query-handling/dispatching-queries.md)
* [Complex business transactions](implementing-domain-logic/complex-business-transactions/complex-business-transactions.md)
    * [Deciding when to use a Saga](implementing-domain-logic/complex-business-transactions/when-to-use-saga.md)
    * [Implementing a Saga](implementing-domain-logic/complex-business-transactions/implementing-saga.md)
    * [Dealing with errors](implementing-domain-logic/complex-business-transactions/_dealing-with-errors.md)
    * [Deadline handling](implementing-domain-logic/complex-business-transactions/deadline-handling.md)
    * [Managing associations](implementing-domain-logic/complex-business-transactions/managing_associations.md)
    * [Testing a Saga](implementing-domain-logic/complex-business-transactions/testing.md)
    <!--    * [Alternatives to Sagas](... explain BPMN engines ...) -->
<!--
* [Connecting the UI](implementing-domain-logic/connecting-the-ui/_connecting-the-ui.md)
    * [Command publishing use cases](implementing-domain-logic/connecting-the-ui/_command-publishing-use-cases.md)
    * [Dealing with eventual consistency](implementing-domain-logic/connecting-the-ui/_dealing-with-eventual-consistency.md)
    * [Query publishing use cases](implementing-domain-logic/connecting-the-ui/_query-publishing-use-cases.md)
-->

## Configuring Infrastructure components

* [Messaging concepts](configuring-infrastructure-components/messaging-concepts/_messaging-concepts.md)
    * [Message Intercepting](configuring-infrastructure-components/messaging-concepts/message-intercepting.md)
    * [Supported Parameters for annotated handlers](configuring-infrastructure-components/messaging-concepts/supported-parameters-for-annotated-handlers.md)
    * [Unit of Work](configuring-infrastructure-components/messaging-concepts/unit-of-work.md)
* [Command processing](configuring-infrastructure-components/command-handling-publishing/command-handling-publishing.md)
    * [Command dispatching](configuring-infrastructure-components/command-handling-publishing/command-dispatching.md)
    * [Command Model Repositories](configuring-infrastructure-components/command-handling-publishing/command_model_repositories.md)
    * [Optimizing Aggregate loading](configuring-infrastructure-components/command-handling-publishing/optimizing_aggregate_loading.md)
* [Event processing](configuring-infrastructure-components/event-processing/event-processing.md)
    * [Event Bus & Event Store](configuring-infrastructure-components/event-processing/event-bus-and-event-store.md)
    * [Event Processors](configuring-infrastructure-components/event-processing/event-processors.md)
* [Query processing](configuring-infrastructure-components/query-processing.md)
* [Deadlines](configuring-infrastructure-components/deadlines.md)

## Production considerations

* [Application versioning](production-considerations/application-versioning/application-versioning.md)
    * [Versioning events](production-considerations/application-versioning/versioning-events.md)
<!--    * [Blue-green deployment approaches](production-considerations/application-versioning/_blue-green-deployment.md)-->
* [Serializers](production-considerations/serializers/serializers.md)
<!--
    * [Custom serializer](production-considerations/serializers/_custom-serializer.md)
    * [Content type converters](production-considerations/serializers/_content-type-converters.md)
-->

## Operations Guide 

* [Setting up Axon Server](operations-guide/setting-up-axon-server/_setting-up-axon-server.md)
    * [Launch](operations-guide/setting-up-axon-server/_launch.md)
    * [Tuning](operations-guide/setting-up-axon-server/_tuning.md)
    * [Backups](operations-guide/setting-up-axon-server/backups.md)
    * [Monitoring](operations-guide/setting-up-axon-server/monitoring.md)
    * [Access Control](operations-guide/setting-up-axon-server/access-control.md)
    * [Axon Server clustering](operations-guide/setting-up-axon-server/_axon-server-clustering.md)
    * [Multi-context](operations-guide/setting-up-axon-server/_multi-context.md)
    * [Field-level encryption (GDPR module)](operations-guide/setting-up-axon-server/_field-level-encryption.md)
    * [Big data & Fast data](operations-guide/setting-up-axon-server/_big-data-fast-data.md)
* [Alternative event storage options](operations-guide/_alternative-event-storage-options.md)
* [Scaling out](operations-guide/scaling-out/_scaling-out.md)
    * [Considerations when scaling](operations-guide/scaling-out/_considerations-when-scaling.md)
    * [Scaling event processors](operations-guide/scaling-out/_scaling-event-processors.md)

## Extensions

* [Spring AMQP](extensions/spring-amqp.md)
* [Kafka](extensions/kafka.md)
* [JGroups](extensions/jgroups.md)
* [Spring Cloud](extensions/spring-cloud.md)
* [Mongo](extensions/mongo.md)

## Appendices

* [A. RDBMS Tuning](appendices/rdbms-tuning.md)

* [B. Tuning Message Handler definitions](appendices/handler-definitions/handler-definitions.md)
    * [Custom Handler Definition](appendices/handler-definitions/_custom-handler-definition.md)
    * [Handler Enhancers](appendices/handler-definitions/_custom-handler-enhancers.md)
