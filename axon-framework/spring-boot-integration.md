# Spring Boot Integration

Axon Framework provides extensive support for Spring, but does not require you to use Spring in order to use Axon. All components can be configured programmatically and do not require Spring on the classpath. However, if you do use Spring, much of the configuration is made easier with the use of Spring's annotation support. Axon provides Spring Boot starters on the top of that, so you can benefit from auto-configuration as well.

## Auto-configuration

Axon's Spring Boot auto-configuration is by far the easiest option to get started configuring your Axon components. By simply declaring dependency to `axon-spring-boot-starter`, Axon will automatically configure the infrastructure components \(command bus, event bus, query bus\), as well as any component required to run and store aggregates and sagas.

## Demystifying Axon Spring Boot Starter

With a lot of things happening in the background, it sometimes becomes difficult to understand how an annotation or just including a dependency enables so many features.

`axon-spring-boot-starter` follows general Spring boot convention in structuring the starter. It depends on `axon-spring-boot-autoconfigure` which holds concrete implementation of Axon auto-configuration. When Axon Spring Boot application starts up, it looks for a file named `spring.factories` in the `classpath`. This file is located in the `META-INF` directory of `axon-spring-boot-autoconfigure` module:

```text
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    org.axonframework.springboot.autoconfig.MetricsAutoConfiguration,\
    org.axonframework.springboot.autoconfig.EventProcessingAutoConfiguration,\
    org.axonframework.springboot.autoconfig.AxonAutoConfiguration,\
    org.axonframework.springboot.autoconfig.JpaAutoConfiguration,\
    org.axonframework.springboot.autoconfig.JpaEventStoreAutoConfiguration,\
    org.axonframework.springboot.autoconfig.JdbcAutoConfiguration,\
    org.axonframework.springboot.autoconfig.TransactionAutoConfiguration,\
    org.axonframework.springboot.autoconfig.NoOpTransactionAutoConfiguration,\
    org.axonframework.springboot.autoconfig.InfraConfiguration,\
    org.axonframework.springboot.autoconfig.ObjectMapperAutoConfiguration,\
    org.axonframework.springboot.autoconfig.AxonServerAutoConfiguration
```

This file maps different configuration classes which Axon Spring boot application will try to apply. So, as per this snippet, Spring Boot will try to apply all the configuration classes for `AxonServerAutoConfiguration`, `AxonAutoConfiguration`, ...

Whether these configuration classes will be applied or not, it will depend on conditions defined on this classes:

* `AxonServerAutoConfiguration` configures Axon Server as implementation for the Command Bus, Query Bus and Event Store. It will be applied before `AxonAutoConfiguration`, and it will be applied only if the `org.axonframework.axonserver.connector.AxonServerConfiguration` class is available in the classpath. Axon Server auto configuration can be disabled by setting the `axon.axonserver.enabled` property to `false` in the `.properties`/`.yml` file.
* `AxonAutoConfiguration` configures a 'non-axon-server' implementation of Command Bus, Query Bus, Event Store/Event Bus and other Axon components. These components will be initialized only if they are not in the Spring Application context already, eg. `@ConditionalOnMissingBean(EventBus.class)`. As `AxonAutoConfiguration` will be applied after `AxonServerAutoConfiguration` these Axon components will be in the Spring Application Context already, and therefore Axon Server's implementation of Command Bus, Query Bus and Event Store/Event Bus will win.

Axon Spring Boot auto-configuration is not intrusive. It will define only Spring components that you haven't already explicitly defined in the application context. This allow you to completely override the auto-configured beans by defining your own in one of the `@Configuration` classes.

Specific Axon \(Spring\) component configurations will be explained in detail in the following sections of this guide.

