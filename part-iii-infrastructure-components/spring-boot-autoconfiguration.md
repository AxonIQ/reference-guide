# Spring Boot AutoConfiguration

Axon's support for Spring Boot AutoConfiguration is by far the easiest option to get started configuring your Axon infrastructure components. By simply adding the `axon-spring-boot-starter` dependency, Axon will automatically configure the basic infrastructure components \(Command Bus, Event Bus\), as well as any component required to run and store Aggregates and Sagas.

Depending on other components available in the application context, Axon will define certain components, if they aren't already explicitly defined in the application context. This means that you only need to configure components that you want different from the default.

## Event Bus and Event Store Configuration

If JPA is available, an Event Store with a JPA Event Storage Engine is used by default. This allow storage of Aggregates using Event Sourcing without any explicit configuration.

If JPA is not available, Axon defaults to a `SimpleEventBus`, which means that you need to specify a non-event sourcing repository for each Aggregate, or configure an `EventStorageEngine` in your Spring Configuration.

To configure a different Event Storage Engine, even if JPA is on the class path, simply define a bean of type `EventStorageEngine` \(to use Event Sourcing\) or `EventBus` \(if Event Sourcing isn't required\).

## Command Bus Configuration

Axon will configure a `SimpleCommandBus` if no `CommandBus` implementation is explicitly defined in the Application Context. This `CommandBus` will use a `PlatformTransactionManager` to manage transactions, if it is available in the context.

If the only `CommandBus` bean defined is a `DistributedCommandBus` implementation, Axon will still configure a CommandBus implementation to serve as the local segment for the DistributedCommandBus. This bean will get a Qualifier "localSegment". It is recommended to define the `DistributedCommandBus` as a `@Primary`, so that it gets priority for dependency injection.

## Aggregate Configuration

The `@Aggregate` annotation \(in package `org.axonframework.spring.stereotype`\) triggers AutoConfiguration to set up the necessary components to use the annotated type as an Aggregate. Note that only the Aggregate Root needs to be annotated.

Axon will automatically register all the `@CommandHandler` annotated methods with the Command Bus and set up a repository if none is present.

To set up a different repository than the default, define one in the application context. Optionally, you may define the name of the repository to use, using the `repository` attribute on `@Aggregate`. If no `repository` attribute is defined, Axon will attempt to use the repository with the name of the aggregate \(first character lowercase\), suffixed with `Repository`. So on a class of type `MyAggregate`, the default Repository name is `myAggregateRepository`. If no bean with that name is found, Axon will define an `EventSourcingRepository` \(which fails if no `EventStore` is available\).

## Saga Configuration

The configuration of infrastructure components to operate Sagas is triggered by the `@Saga` annotation \(in package `org.axonframework.spring.stereotype`\). Axon will configure a `SagaManager` and `SagaRepository`. The SagaRepository will use a `SagaStore` available in the context \(defaulting to `JPASagaStore` if JPA is found\) for the actual storage of Sagas.

To use different `SagaStore`s for Sagas, provide the bean name of the `SagaStore` to use in the `sagaStore` attribute of each `@Saga` annotation.

Sagas will have resources injected from the application context. Note that this doesn't mean Spring-injecting is used to inject these resources. The `@Autowired` and `@javax.inject.Inject` annotation can be used to demarcate dependencies, but they are injected by Axon by looking for these annotations on Fields and Methods. Constructor injection is not \(yet\) supported.

## Event Handling Configuration

By default, all singleton Spring beans components containing `@EventHandler` annotated methods will be subscribed to an Event Processor to receive Event Messages published to the Event Bus.

The `EventHandlingConfiguration` bean, available in the Application Context, has methods to tweak the configuration of the Event Handlers. See [Configuration API](../part-i-getting-started/configuration-api.md) for details on configuring Event Handlers and Event Processors.

To update the Event Handling Configuration, create an autowired method that set the configuration you desire:

```java
@Autowired
public void configure(EventHandlingConfiguration config) {
    config.usingTrackingProcessors(); // default all processors to tracking mode.
}
```

Certain aspect of Event Processors can also be configured in `application.properties`.

```text
axon.eventhandling.processors["name"].mode=tracking
axon.eventhandling.processors["name"].source=eventBus
```

Of using application.yml:

```yaml
axon:
    eventhandling:
        processors:
            name:
                mode: tracking
                source: eventBus
```

The source attribute refers to the name of a bean implementing `SubscribableMessageSource` or `StreamableMessageSource` that should be used as the source of events for the mentioned processor. The source default to the Event Bus or Event Store defined in the application context.

## Enabling AMQP

To enable AMQP support, ensure that the `axon-amqp` module is on the classpath and an AMQP `ConnectionFactory` is available in the application context \(e.g. by including the `spring-boot-starter-amqp`\).

To forward Events generated in the application to an AMQP Channel, a single line of `application.properties` configuration is sufficient:

```text
axon.amqp.exchange=ExchangeName
```

This will automatically send all published events to the AMQP Exchange with the given name.

By default, no AMQP transactions are used when sending. This can be overridden using the `axon.amqp.transaction-mode` property, and setting it to `transactional` or `publisher-ack`.

To receive events from a queue and process them inside an Axon application, you need to configure a `SpringAMQPMessageSource`:

```java
@Bean
public SpringAMQPMessageSource myQueueMessageSource(AMQPMessageConverter messageConverter) {
    return new SpringAMQPMessageSource(messageConverter) {

        @RabbitListener(queues = "myQueue")
        @Override
        public void onMessage(Message message, Channel channel) throws Exception {
            super.onMessage(message, channel);
        }
    };
}
```

and then configure a processor to use this bean as the source for its messages:

```text
axon.eventhandling.processors["name"].source=myQueueMessageSource
```

## Distributing commands

Configuring a distributed command bus can \(mostly\) be done without any modifications in Configuration files.

First of all, the starters for one of the Axon Distributed Command Bus modules needs to be included \(e.g. JGroups or SpringCloud\).

Once that is present, a single property needs to be added to the application context, to enable the distributed command bus:

```text
axon.distributed.enabled=true
```

There in one setting that is independent of the type of connector used:

```text
axon.distributed.load-factor=100
```

Axon will automatically configure a DistributedCommandBus when a `CommandRouter` as well as a `CommandBusConnector` are present in the application context. In such case, specifying `axon.distributed.enabled` isn't even necessary. The latter merely enables autoconfiguration of these routers and connectors.

### Using JGroups

This module uses JGroups to detect and communicate with other nodes. The AutoConfiguration will set up the JGroupsConnector using default settings, that may need to be adapted to suit your environment.

By default, the JGroupsConnector will attempt to locate a GossipRouter on the localhost, port 12001.

The settings for the JGroups connector are all prefixed with `axon.distributed.jgroups`.

```text
# the address to bind this instance to. By default, attempts to find the Global IP address
axon.distributed.jgroups.bind-addr=GLOBAL
# the port to bind the local instance to
axon.distributed.jgroups.bind-port=7800

# the name of the JGroups Cluster to connect to
axon.distributed.jgroups.cluster-name=Axon

# the JGroups Configuration file to configure JGroups with
axon.distributed.jgroups.configuration-file=default_tcp_gossip.xml

# The IP and port of the Gossip Servers (comma separated) to connect to
axon.distributed.jgroups.gossip.hosts=localhost[12001]
# when true, will start an embedded Gossip Server on bound to the port of the first mentioned gossip host.
axon.distributed.jgroups.gossip.auto-start=false
```

The JGroups Configuration file can be use for much more fine-grained control of the Connector's behavior. Check out JGroups's Reference Guide for more information.

### Using Spring Cloud

Spring Cloud comes with nice abstractions on top of discovery. Axon can use these abstractions to report its availability and find other Command Bus nodes. For communication with these nodes, Axon uses Spring HTTP, by default.

The Spring Cloud AutoConfiguration doesn't have much to configure. It uses an existing Spring Cloud Discovery Client \(so make sure `@EnableDiscoveryClient` is used and the necessary client is on the classpath\).

However, some discovery clients aren't able to update instance metadata dynamically on the server. If Axon detects this, it will automatically fall back to querying that node using HTTP. This is done once on each discovery heartbeat \(usually 30 seconds\).

This behavior can be configured or disabled, using the following settings in `appplication.properties`:

```text
# whether to fall back to http when no meta-data is available
axon.distributed.spring-cloud.fallback-to-http-get=true
# the URL on which to publish local data and retrieve from other nodes.
axon.distributed.spring-cloud.fallback-url=/message-routing-information
```

For more fine-grained control, provide a `SpringCloudHttpBackupCommandRouter` or `SpringCloudCommandRouter` bean in your application context.

