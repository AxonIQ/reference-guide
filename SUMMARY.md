# Table of contents

* [Quick start](introduction/_quick-start.md)
* [Architecture overview](introduction/architecture-overview.md)
* [Event driven design](introduction/_event-driven-design.md)

## Setting up

* [Prerequisites](setting-up/_prerequisites.md)
* [Starting the Axon server](setting-up/starting-the-axon-server.md)
* [Maven/Gradle dependencies](setting-up/maven-dependencies.md)
* [Spring boot solution](setting-up/spring-boot.md)
* [Java CDI solution](setting-up/_java-cdi.md)
* [Code structure considerations](setting-up/_code-structure.md)

## Implementing domain logic

* [Command model](implementing-domain-logic/command-model/command-model.md)
    * [Aggregate](implementing-domain-logic/command-model/aggregate.md)
    * [External command handler](implementing-domain-logic/command-model/external-command-handler.md)
    * [Multi-entity aggregates](implementing-domain-logic/command-model/multi-entity-aggregates.md)
    * [State-stored aggregates](implementing-domain-logic/command-model/state-stored-aggregates.md)
    * [Dispatching commands](implementing-domain-logic/command-model/dispatching-commands.md)
    * [Testing](implementing-domain-logic/command-model/testing.md)
* [Event handling](implementing-domain-logic/event-handling/event-handling.md)
    * [Handling events](implementing-domain-logic/event-handling/handling-events.md)
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
* [Connecting the UI](implementing-domain-logic/connecting-the-ui/_connecting-the-ui.md)
    * [Command publishing use cases](implementing-domain-logic/connecting-the-ui/_command-publishing-use-cases.md)
    * [Dealing with eventual consistency](implementing-domain-logic/connecting-the-ui/_dealing-with-eventual-consistency.md)
    * [Query publishing use cases](implementing-domain-logic/connecting-the-ui/_query-publishing-use-cases.md)

## Configuring Infrastructure components

* [Messaging concepts](configuring-infrastructure-components/_messaging-concepts.md)
* [Command handling/publishing](configuring-infrastructure-components/command-handling-publishing/command-handling-publishing.md)
    * [Command dispatching](configuring-infrastructure-components/command-handling-publishing/command-dispatching.md)
    * [Command Model Repositories](configuring-infrastructure-components/command-handling-publishing/command_model_repositories.md)
    * [Optimizing aggregate loading](configuring-infrastructure-components/command-handling-publishing/optimizing_aggregate_loading.md)
* [Event processing](configuring-infrastructure-components/event-processing/event-processing.md)
    * [Event bus & Event store](configuring-infrastructure-components/event-processing/event-bus-and-event-store.md)
    * [Event processors](configuring-infrastructure-components/event-processing/event-processors.md)
* [Query bus](configuring-infrastructure-components/query-bus.md)
* [Scheduling deadlines](configuring-infrastructure-components/scheduling-deadlines.md)
* [Message Intercepting](configuring-infrastructure-components/message-intercepting.md)
* [Supported Parameters for annotated handlers](configuring-infrastructure-components/supported-parameters-for-annotated-handlers.md)
* [UnitOfWork](configuring-infrastructure-components/unit-of-work.md)

## Production considerations

* [Application versioning](production-considerations/application-versioning/application-versioning.md)
    * [Versioning events](production-considerations/application-versioning/versioning-events.md)
    * [Blue-green deployment approaches](production-considerations/application-versioning/_blue-green-deployment.md)
* [Serializers](production-considerations/serializers.md)

## Operations Guide 

* [Setting up Axon Server](operations-guide/setting-up-axon-server/_setting-up-axon-server.md)
    * [Launch](operations-guide/setting-up-axon-server/_launch.md)
    * [Tuning](operations-guide/setting-up-axon-server/_tuning.md)
    * [Clustering](operations-guide/setting-up-axon-server/_clustering.md)
* [Alternative event storage options](operations-guide/_alternative-event-storage-options.md)
* [Scaling out](operations-guide/scaling-out/_scaling-out.md)
    * [Considerations when scaling](operations-guide/scaling-out/_considerations-when-scaling.md)
    * [Scaling event processors](operations-guide/scaling-out/_scaling-event-processors.md)
* [Axon Server clustering](operations-guide/_axon-server-clustering.md)
* [Multi-context](operations-guide/_multi-context.md)
* [Field-level encryption (GDPR module)](operations-guide/_field-level-encryption.md)
* [Big data & Fast data](operations-guide/_big-data-fast-data.md)

## Advanced configuration

* [Handler definition](advanced-configuration/_handler-definition.md)
* [Handler enhancers](advanced-configuration/_handler-enhancers.md)
* [Content type converters](advanced-configuration/_content-type-converters.md)

## Extensions

* [Spring AMQP](extensions/spring-amqp.md)
* [Kafka](extensions/kafka.md)
* [JGroups](extensions/jgroups.md)
* [Spring Cloud](extensions/spring-cloud.md)
* [Mongo](extensions/mongo.md)
