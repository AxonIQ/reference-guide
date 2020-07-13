# Spring Cloud

Spring Cloud is an alternative approach to distributing command bus \(commands\), besides Axon Server which is the default.

The Spring Cloud connector setup uses the service registration and discovery mechanism described by [Spring Cloud](https://spring.io/projects/spring-cloud) for distributing the command bus. You are thus left free to choose which Spring Cloud implementation to use to distribute your commands. An example implementations is the Eureka Discovery/Eureka Server combination.

To use the Spring Cloud components from Axon, make sure the `axon-springcloud` module is available on the classpath.

> **Note**
>
> The `SpringCloudCommandRouter` uses the Spring Cloud specific `ServiceInstance.Metadata` field to inform all the nodes in the system of its message routing information. It is thus of importance that the Spring Cloud implementation selected supports the usage of the `ServiceInstance.Metadata` field. If the desired Spring Cloud implementation does not support the modification of the `ServiceInstance.Metadata` \(e.g. Consul\), the `SpringCloudHttpBackupCommandRouter` is a viable solution. See the end of this chapter for configuration specifics on the `SpringCloudHttpBackupCommandRouter`.

Giving a description of every Spring Cloud implementation would push this reference guide to far. Hence we refer to their respective documentations for further information.

The Spring Cloud connector setup is a combination of the `SpringCloudCommandRouter` and a `SpringHttpCommandBusConnector`, which respectively fill the place of the `CommandRouter` and the `CommandBusConnector` for the `DistributedCommandBus`.

> **Note**
>
> When using the `SpringCloudCommandRouter`, make sure that your Spring application is has heartbeat events enabled. The implementation leverages the heartbeat events published by a Spring Cloud application to check whether its knowledge of all the others nodes is up to date. If heartbeat events are disabled, then the majority of the Axon applications within your cluster will not be aware of the entire setup. This will cause issues for correct command routing.

The `SpringCloudCommandRouter` has to be created by providing the following:

* A "discovery client" of type `DiscoveryClient` - This can be provided by annotating your Spring Boot application with `@EnableDiscoveryClient`, which will look for a Spring Cloud implementation on your classpath.
* A "routing strategy" of type `RoutingStrategy` - The `axon-messaging` module currently provides several implementations, but a function call can suffice as well.

  If you want to route the commands based on the 'aggregate identifier' for example, you would use the `AnnotationRoutingStrategy` and annotate the field on the payload that identifies the aggregate with `@TargetAggregateIdentifier`.

* A "local service instance" of type `Registration` - If you're Spring Boot application is annotated with the aforementioned `@EnableDiscoveryClient`, it will automatically create a `Registration` bean referencing the instance itself.

Other optional parameters for the `SpringCloudCommandRouter` are:

* A "service instance filter" of type `Predicate<ServiceInstance>` - This predicate is used to filter out `ServiceInstances` which the `DiscoveryClient` might encounter which by forehand you know will not handle any command messages. This might be useful if you've got several services within the Spring Cloud Discovery Service set up which you do not want to take into account for command handling, ever.
* A "consistent hash change listener" of type `ConsistentHashChangeListener` - Adding a consistent hash change listener provides you the opportunity to perform a specific task if new members have been added to the known command handlers set.

The `SpringHttpCommandBusConnector` requires three parameters for creation:

* A "local command bus" of type `CommandBus` - This is the command bus implementation that dispatches commands to the local JVM. These commands may have been dispatched by instances on other JVMs or from the local one.
* A `RestOperations` object to perform the posting of a command message to another instance.
* Lastly a "serializer" of type `Serializer` - The serializer is used to serialize the command messages before they are sent over the wire.

> **Note**
>
> The Spring Cloud connector specific components for the `DistributedCommandBus` can be found in the `axon-distributed-commandbus-springcloud` module.

The `SpringCloudCommandRouter` and `SpringHttpCommandBusConnector` should then both be used for creating the `DistributedCommandBus`. In Spring Java config, that would look as follows:

```java
// Simple Spring Boot App providing the `DiscoveryClient` bean
@EnableDiscoveryClient
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    // Example function providing a Spring Cloud Connector
    @Bean
    public CommandRouter springCloudCommandRouter(DiscoveryClient discoveryClient, Registration localServiceInstance) {
        return SpringCloudCommandRouter.builder()
                                       .discoveryClient(discoveryClient)
                                       .routingStrategy(new AnnotationRoutingStrategy())
                                       .localServiceInstance(localServiceInstance)
                                       .build();
    }

    @Bean
    public CommandBusConnector springHttpCommandBusConnector(
                        @Qualifier("localSegment") CommandBus localSegment,
                        RestOperations restOperations,
                        Serializer serializer) {
        return SpringHttpCommandBusConnector.builder()
                                            .localCommandBus(localSegment)
                                            .restOperations(restOperations)
                                            .serializer(serializer)
                                            .build();
    }

    @Primary // to make sure this CommandBus implementation is used for autowiring
    @Bean
    public DistributedCommandBus springCloudDistributedCommandBus(
                         CommandRouter commandRouter, 
                         CommandBusConnector commandBusConnector) {
        return DistributedCommandBus.builder()
                                    .commandRouter(commandRouter)
                                    .connector(commandBusConnector)
                                    .build();
    }

}
```

```java
// if you don't use Spring Boot Autoconfiguration, you will need to explicitly define the local segment:
@Bean
@Qualifier("localSegment")
public CommandBus localSegment() {
    return SimpleCommandBus.builder().build();
}
```

> **Note**
>
> Note that it is not required that all segments have command handlers for the same type of commands. You may use different segments for different command types altogether. The Distributed Command Bus will always choose a node to dispatch a command to that has support for that specific type of command.

**Spring Cloud Http Back Up Command Router**

Internally, the `SpringCloudCommandRouter` uses the `Metadata` map contained in the Spring Cloud `ServiceInstance` to communicate the allowed message routing information throughout the distributed Axon environment. However, if the desired Spring Cloud implementation does not allow the modification of the `ServiceInstance.Metadata` field \(e.g. Consul\), one can choose to instantiate a `SpringCloudHttpBackupCommandRouter` instead of the `SpringCloudCommandRouter`.

The `SpringCloudHttpBackupCommandRouter`, as the name suggests, has a backup mechanism if the `ServiceInstance.Metadata` field does not contain the expected message routing information. That backup mechanism is to provide an HTTP endpoint from which the message routing information can be retrieved and by simultaneously adding the functionality to query that endpoint of other known nodes in the cluster to retrieve their message routing information. As such, the backup mechanism functions as a Spring Controller to receive requests at a specifiable endpoint and uses a `RestTemplate` to send request to other nodes at the specifiable endpoint.

To use the `SpringCloudHttpBackupCommandRouter` instead of the `SpringCloudCommandRouter`, add the following Spring Java configuration \(which replaces the `SpringCloudCommandRouter` method in our earlier example\):

```java
@Configuration
public class MyApplicationConfiguration {
    @Bean
    public CommandRouter springCloudHttpBackupCommandRouter(
                             DiscoveryClient discoveryClient, 
                             RestTemplate restTemplate,
                             Registration localServiceInstance,                             
                             @Value("${axon.distributed.spring-cloud.fallback-url}") 
                                         String messageRoutingInformationEndpoint) {
        return SpringCloudHttpBackupCommandRouter.builder()
                                                 .discoveryClient(discoveryClient)
                                                 .routingStrategy(new AnnotationRoutingStrategy())
                                                 .restTemplate(restTemplate)
                                                 .localServiceInstance(localServiceInstance)
                                                 .messageRoutingInformationEndpoint(messageRoutingInformationEndpoint)
                                                 .build();
    }
}
```

## Configuration in Spring Boot

The Spring Cloud Auto-configuration doesn't have much to configure. It uses an existing Spring Cloud Discovery Client \(so make sure `@EnableDiscoveryClient` is used and the necessary client is on the classpath\).

However, some discovery clients are not able to update instance metadata dynamically on the server. If Axon detects this, it will automatically fall back to querying that node using HTTP. This is done once on each discovery heartbeat \(usually 30 seconds\).

This behavior can be configured or disabled using the following settings in `appplication.properties`:

```text
# whether to fall back to http when no meta-data is available
axon.distributed.spring-cloud.fallback-to-http-get=true
# the URL on which to publish local data and retrieve from other nodes.
axon.distributed.spring-cloud.fallback-url=/message-routing-information
```

