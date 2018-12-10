# Spring (boot)

Axon Framework provides extensive support for Spring, but does not require you to use Spring in order to use Axon. All components can be configured programmatically and do not require Spring on the classpath. However, if you do use Spring, much of the configuration is made easier with the use of Spring's annotation support. Axon provides Spring Boot starters on top of that, so you can benefit from auto-configuration as well.

## Auto-configuration

Axon's Spring Boot auto-configuration is by far the easiest option to get started configuring your Axon components. By simply adding the `axon-spring-boot-starter` dependency, Axon will automatically configure the basic infrastructure components \(command bus, event bus, query bus\), as well as any component required to run and store aggregates and sagas.

By include this dependencies you will be on the way of creating web application very quickly:

```
<properties>
        <axon.version>4.0.3</axon.version>
</properties>
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.axonframework</groupId>
        <artifactId>axon-spring-boot-starter</artifactId>
        <version>${axon.version}</version>
    </dependency>

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.axonframework</groupId>
        <artifactId>axon-test</artifactId>
        <version>${axon.version}</version>
        <scope>test</scope>
    </dependency>
    ...
</dependencies>
```


## Advanced Spring configuration

Axon Spring Boot auto-configuration is not intrusive. It will define only Spring components that you aren't already explicitly defined in the application context. This means that you only need to explicitly configure components that you want different from the default/convention. As you make progress and implement your components you will probably find a reason to do that, in that case refer to this section to find out how.

### Event bus and event store configuration (TODO rewrite-axon server default)

If JPA is available, an event store with a `JpaEventStorageEngine` is used by default. This allow storage of Aggregates using Event Sourcing without any explicit configuration.

If JPA is not available, Axon defaults to a `SimpleEventBus`, which means that you need to specify a non-event sourcing repository for each aggregate, or configure an `EventStorageEngine` in your Spring configuration.

To configure a different event storage engine, even if JPA is on the class path, simply define a bean of type `EventStorageEngine` \(to use event sourcing\) or `EventBus` \(if event sourcing is not required\).

### Command bus configuration (TODO rewrite-axon server default)

Axon will configure a `SimpleCommandBus` if no `CommandBus` implementation is explicitly defined in the Application Context. This `CommandBus` will use the `TransactionManager` to manage transactions.

If the only `CommandBus` bean defined is a `DistributedCommandBus` implementation, Axon will still configure a `CommandBus` implementation to serve as the local segment for the `DistributedCommandBus`. This bean will get a Qualifier `"localSegment"`. It is recommended to define the `DistributedCommandBus` as a `@Primary`, so that it gets priority for dependency injection.

### Query bus configuration (TODO rewrite-axon server default)

Axon will configure a `SimpleQueryBus` if no `QueryBus` implementation is explicitly defined in the Application Context. This `QueryBus` will use the `TransactionManager` to manage transactions.

### Transaction manager configuration

If no `TransactionManager` implementation is explicitly defined in the application context, Axon will look for the Spring `PlatformTransactionManager` bean and wrap that in a `TransactionManager`. If the Spring bean is not available, the `NoOpTransactionManager` will be used.

### Serializer configuration

By default, Axon uses an XStream based serializer to serialize objects, as is described in further detail in the [Advanced Customizations](../1.4-advanced-tuning/advanced-customizations.md#serializers) section. This can be changed by defining a bean of type `Serializer` in the application context.

While the default Serializer provides an, arguably ugly, XML based format, it is capable of serializing and deserializing virtually anything, making it a very sensible default. However, for events, which needs to be stored for a long time and perhaps shared across application boundaries, it is desirable to customize the format.

You can define a separate `Serializer` to be used to serialize events, by assigning it the `eventSerializer` qualifier. Axon will consider a bean with this qualifier to be the event serializer. If no other bean is defined, Axon will use the default serializer for all other objects to serialize.

Example:

```java
class SerializerConfiguration {

    @Qualifier("eventSerializer")
    @Bean
    public Serializer eventSerializer() {
        return new JacksonSerializer();
    }
}
```

Equal to events, you can also customize the Message `Serializer` used by your application. The Message `Serializer` comes into play when your command and query message are sent from one node to another in a distributed environment. To set a custom `Serializer` for you message you can simply define a `messageSerializer` bean like so:

```java
class SerializerConfiguration {

    @Qualifier("messageSerializer")
    @Bean
    public Serializer messageSerializer() {
        return new XStreamSerializer();
    }
}
```

When overriding both the default serializer and defining an event serializer, we must instruct Spring that the default serializer is, well, the default:

```java
class SerializerConfiguration {

    @Primary // marking it primary means this is the one to use, 
             // if no specific Qualifier is requested
    @Bean
    public Serializer serializer() {
        return new MyCustomSerializer();
    }

    @Qualifier("eventSerializer")
    @Bean
    public Serializer eventSerializer() {
        return new JacksonSerializer();
    }
}
```

### Aggregate configuration

The `@Aggregate` annotation \(in package `org.axonframework.spring.stereotype`\) triggers auto configuration to set up the necessary components to use the annotated type as an aggregate. Note that only the aggregate root needs to be annotated.

Axon will automatically register all the `@CommandHandler` annotated methods with the command bus and set up a repository if none is present.

It is possible to define a custom [`SnapshotTriggerDefinition`](repository-and-event-store.md#creating-a-snapshot) for an aggregate as a spring bean. In order to tie the `SnapshotTriggerDefinition` bean to an aggregate, use the `snapshotTriggerDefinition` attribute on `@Aggregate` annotation. Listing below shows how to define a custom `EventCountSnapshotTriggerDefinition` which will take a snapshot on each five hundredths event.

Note that a `Snapshotter` instance, if not explicitly defined as a bean already, will be automatically configured for you. This means you can simply pass the `Snapshotter` as a parameter to your `SnapshotTriggerDefinition`.

```java
@Bean
public SnapshotTriggerDefinition mySnapshotTriggerDefinition(
                                                  Snapshotter snapshotter) {
    return new EventCountSnapshotTriggerDefinition(snapshotter, 500);
}

...

@Aggregate(snapshotTriggerDefinition = "mySnapshotTriggerDefinition")
public class MyAggregate {...}
```

Defining a [`CommandTargetResolver`](../1.2-domain-logic/command-model.md#handling-commands-in-an-aggregate) as a bean in the Spring application context will cause that resolver to be used for all aggregate definitions. However, you can also define multiple beans and specify the instance to use with the `commandTargetResolver` attribute on `@Aggregate` annotation will override this behavior. You can for example define a `MetaDataCommandTargetResolver` which will look for `myAggregateId` key in metadata is listed below together with assignment to the aggregate.

```java
@Bean
public CommandTargetResolver myCommandTargetResolver() {
    return new MetaDataCommandTargetResolver("myAggregateId");
}
...
@Aggregate(commandTargetResolver = "myCommandTargetResolver")
public class MyAggregate {...}
```

To fully customize the repository used, you can define one in the application context. For Axon Framework to use this repository for the intended aggregate, define the bean name of the repository in the `repository` attribute on `@Aggregate` Annotation. Alternatively, specify the bean name of the repository to be the aggregate's name, \(first character lowercase\), suffixed with `Repository`. So on a class of type `MyAggregate`, the default repository name is `myAggregateRepository`. If no bean with that name is found, Axon will define an `EventSourcingRepository` \(which fails if no `EventStore` is available\).

```java
@Bean
public Repository<MyAggregate> repositoryForMyAggregate(EventStore eventStore) {
    return new EventSourcingRepository<>(MyAggregate.class, eventStore);
}
...
@Aggregate(repository = "repositoryForMyAggregate")
public class MyAggregate {...}
```

Note that this requires full configuration of the Repository, including any `SnapshotTriggerDefinition` or `AggregateFactory` that may otherwise have been configured automatically.

## Saga configuration

The configuration of infrastructure components to operate sagas is triggered by the `@Saga` annotation \(in package `org.axonframework.spring.stereotype`\). Axon will configure a `SagaManager` and `SagaRepository`. The `SagaRepository` will use a `SagaStore` available in the context \(defaulting to `JPASagaStore` if JPA is found\) for the actual storage of sagas.

To use different `SagaStore`s for sagas, provide the bean name of the `SagaStore` to use in the `sagaStore` attribute of each `@Saga` annotation.

Sagas will have resources injected from the application context. Note that this does not mean Spring-injecting is used to inject these resources. The `@Autowired` and `@javax.inject.Inject` annotation can be used to demarcate dependencies, but they are injected by Axon by looking for these annotations on fields and methods. Constructor injection is not \(yet\) supported.

To tune the configuration of sagas, it is possible to define a custom `SagaConfiguration` bean. For an annotated saga class, Axon will attempt to find a configuration for that saga. It does so by checking for a bean of type `SagaConfiguration` with a specific name. For a saga class called `MySaga`, the bean that Axon looks for is `mySagaConfiguration`. If no such bean is found, it creates a configuration based on available components.

If a `SagaConfiguration` instance is present for an annotated saga, that configuration is used to retrieve and register the components for this type of saga. If the `SagaConfiguration` bean is not named as described above, it is possible that the saga is registered twice, and receives events in duplicate. To prevent this, you can specify the bean name of the `SagaConfiguration` using the `@Saga` annotation:

```java
@Saga(configurationBean = "mySagaConfigBean")
public class MySaga {
    // methods here 
}

// in the Spring configuration:
@Bean 
public SagaConfiguration<MySaga> mySagaConfigurationBean() {
    // create and return SagaConfiguration instance
}
```

### Event handling configuration

By default, all singleton Spring beans components containing `@EventHandler` annotated methods will be subscribed to an event processor to receive event messages published to the event bus.

The `EventHandlingConfiguration` bean, available in the application context, has methods to tweak the configuration of the event handlers. See [Configuration API](../1.1-concepts/configuration-api.md) for details on configuring event handlers and event processors.

To update the event handling configuration, create an autowired method that set the configuration you desire:

```java
@Autowired
public void configure(EventHandlingConfiguration config) {
    config.usingTrackingProcessors(); // default all processors to tracking mode.
}
```

Certain aspect of event processors can also be configured in `application.properties`.

```text
axon.eventhandling.processors.name.mode=tracking
axon.eventhandling.processors.name.source=eventBus
```

If the name of a processor contains periods `.`, use the map notation:

```text
axon.eventhandling.processors[name].mode=tracking
axon.eventhandling.processors[name].source=eventBus
```

Or using application.yml:

```yaml
axon:
    eventhandling:
        processors:
            name:
                mode: tracking
                source: eventBus
```

The source attribute refers to the name of a bean implementing `SubscribableMessageSource` or `StreamableMessageSource` that should be used as the source of events for the mentioned processor. The source default to the event bus or event store defined in the application context.

### Query handling configuration

All singleton Spring beans are scanned for methods that have the `@QueryHandler` annotation. For each method that is found, a new query handler is registered with the query bus.

#### Parallel processing

Tracking event processors can use multiple threads to process events in parallel. Not all threads need to run on the same node.

One can configure the number of threads \(on this instance\) as well as the initial number of segments that a processor should define, if non are yet available.

```text
axon.eventhandling.processors.name.mode=tracking
# Sets the number of maximum number threads to start on this node
axon.eventhandling.processors.name.threadCount=2
# Sets the initial number of segments (i.e. defines the maximum number of overall threads)
axon.eventhandling.processors.name.initialSegmentCount=4
```

### Enabling AMQP

To enable AMQP support, ensure that the `axon-amqp` module is on the classpath and an AMQP `ConnectionFactory` is available in the application context \(e.g. by including the `spring-boot-starter-amqp`\).

To forward events generated in the application to an AMQP Channel, a single line of `application.properties` configuration is sufficient:

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
axon.eventhandling.processors.name.source=myQueueMessageSource
```

### Distributing commands

Configuring a distributed command bus can \(mostly\) be done without any modifications in configuration files.

First of all, the starters for one of the Axon distributed command bus modules needs to be included \(e.g. JGroups or Spring Cloud\).

Once that is present, a single property needs to be added to the application context, to enable the distributed command bus:

```text
axon.distributed.enabled=true
```

There in one setting that is independent of the type of connector used:

```text
axon.distributed.load-factor=100
```

Axon will automatically configure a `DistributedCommandBus` when a `CommandRouter` as well as a `CommandBusConnector` are present in the application context. In such case, specifying `axon.distributed.enabled` isn't even necessary. The latter merely enables autoconfiguration of these routers and connectors.

#### Using JGroups

This module uses JGroups to detect and communicate with other nodes. The Auto-configuration will set up the JGroupsConnector using default settings, that may need to be adapted to suit your environment.

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

The JGroups configuration file can be use for much more fine-grained control of the connector's behavior. Check out JGroups's reference guide for more information.

#### Using Spring Cloud

Spring Cloud comes with nice abstractions on top of discovery. Axon can use these abstractions to report its availability and find other command bus nodes. For communication with these nodes, Axon uses Spring HTTP, by default.

The Spring Cloud Auto-configuration doesn't have much to configure. It uses an existing Spring Cloud Discovery Client \(so make sure `@EnableDiscoveryClient` is used and the necessary client is on the classpath\).

However, some discovery clients are not able to update instance metadata dynamically on the server. If Axon detects this, it will automatically fall back to querying that node using HTTP. This is done once on each discovery heartbeat \(usually 30 seconds\).

This behavior can be configured or disabled, using the following settings in `appplication.properties`:

```text
# whether to fall back to http when no meta-data is available
axon.distributed.spring-cloud.fallback-to-http-get=true
# the URL on which to publish local data and retrieve from other nodes.
axon.distributed.spring-cloud.fallback-url=/message-routing-information
```

For more fine-grained control, provide a `SpringCloudHttpBackupCommandRouter` or `SpringCloudCommandRouter` bean in your application context.

##### Blacklisting

On each heartbeat the memberships of all the nodes in the cluster are updated. If message routing information of a given service instance is not available on this heartbeat signal, that specific service instance gets blacklisted. That blacklisted service instance will be removed from the blacklist if it is no longer present on thereon following heartbeats.

> **Note**
>
> It is regarded as good practice to assign a random value to every service instance name. In doing so, if a given service instance is restarted, it will receive a different name which will mitigate unnecessary blacklisting of nodes.



