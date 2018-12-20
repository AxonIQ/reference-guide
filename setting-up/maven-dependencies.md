# Maven/Gradle dependencies

Axon Framework consists of a number of modules that target specific problem areas. Depending on the exact needs of your project, you will need to include one or more of these modules.

There are currently two ways of obtaining the module binaries: either download the binaries from our website or preferably configure a repository for your build system ([Maven](http://maven.apache.org/), [Gradle](https://gradle.org/)).

Axon modules are available on [Maven Central](https://search.maven.org/search?q=axonframework).

## Main modules

Axon main modules are the modules that have been thoroughly tested and are robust enough to use in demanding production environments. The maven `groupId` of all these modules is `org.axonframework`. Visit [Maven Central Repository](https://search.maven.org/search?q=g:org.axonframework) to copy coordinates for the version you neeed.

| Module                                                | Artifact Id              | Group Id          | Maven Central                                                             |
| ----------------------------------------------------- | ------------------------ | ----------------- |:-------------------------------------------------------------------------:|
| [Axon Test](#axon-test)                               | axon-test                | org.axonframework | [available](https://search.maven.org/search?q=a:axon-test)                |
| [Axon Server Connector](#axon-server-connector)       | axon-server-connector    | org.axonframework | [available](https://search.maven.org/search?q=a:axon-server-connector)    |
| [Axon Configuration](#axon-configuration)             | axon-configuration       | org.axonframework | [available](https://search.maven.org/search?q=a:axon-configuration)       |
| [Axon Spring](#axon-spring)                           | axon-spring              | org.axonframework | [available](https://search.maven.org/search?q=a:axon-spring)              |
| [Axon Spring Boot Starter](#axon-spring-boot-starter) | axon-spring-boot-starter | org.axonframework | [available](https://search.maven.org/search?q=a:axon-spring-boot-starter) |
| [Axon Messaging](#axon-messaging)                     | axon-messaging           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-messaging)           |
| [Axon Modeling](#axon-modeling)                       | axon-modelling           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-modelling)           |
| [Axon Eventsourcing](#axon-eventsourcing)             | axon-eventsourcing       | org.axonframework | [available](https://search.maven.org/search?q=a:axon-eventsourcing)       |
| [Axon Disruptor](#axon-disruptor)                     | axon-disruptor           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-disruptor)           |
| [Axon Metrics](#axon-metrics)                         | axon-metrics             | org.axonframework | [available](https://search.maven.org/search?q=a:axon-metrics)             |
| [Axon CDI](#axon-cdi)                                 | axon-cdi                 | org.axonframework | [available](https://search.maven.org/search?q=a:axon-cdi)                 |
| [Axon Legacy](#axon-legacy)                           | axon-legacy              | org.axonframework | [available](https://search.maven.org/search?q=a:axon-legacy)              |

### axon-test
This module contains test fixtures that you can use to test Axon based components, such as your Command Handlers, Aggregates and Sagas. You typically do not need this module at runtime and will only need to be added to the classpath for running tests.

### axon-server-connector 
This module provides infrastructure components that connect to AxonServer.

### axon-configuration
This module contains all the necessary components to configure an Axon application.

### axon-spring 
This module allows Axon Framework components to be configured in the Spring Application context. It also provides a number of building block implementations specific to Spring Framework, such as an adapter for publishing and retrieving Axon Events on a Spring Messaging Channel.

### axon-spring-boot-starter
This module provides Spring Boot starters on top of the `axon-spring` module, so you can benefit from Spring auto-configuration.

### axon-messaging 
This module contains all necessary components and building blocks to support command, event and query messaging.

### axon-modelling
This module contains the necessary components to create domain models, like Aggregates and Sagas.

### axon-eventsourcing 
This module contains all necessary infrastructure components to support Event Sourcing Command and Query models.

### axon-disruptor
This module contains a specific CommandBus and Command Handling solution based on the Disruptor paradigm.

### axon-metrics 
This module provides basic implementations based on Coda Hale to collect the monitoring information.

### axon-cdi 
This module provides support for Contexts and Dependency Injection (CDI) for the Java EE platform.

### axon-legacy
This module contains components that enable migration of older Axon projects to use the latest Axon version.

## Extension modules

Besides main modules, there are several extension modules which complement Axon Framework. They address distribution concerns of Axon Framework towards non-Axon Server solutions. The maven `groupId` of all these extensions starts with `org.axonframework.extensions.*`. Visit [Maven Central Repository](https://search.maven.org/search?q=axonframework%20extensions) to copy coordinates for the version you neeed.

| Module                                | Artifact Id                          | Group Id                                 | Maven Central                                                                         |
| ------------------------------------- | ------------------------------------ | ---------------------------------------- |:-------------------------------------------------------------------------------------:|
| Axon AMQP                             | axon-amqp                            | org.axonframework.extensions.amqp        | [available](https://search.maven.org/search?q=a:axon-amqp)                            |
| Axon AMQP Spring Boot Starter         | axon-amqp-spring-boot-starter        | org.axonframework.extensions.amqp        | [available](https://search.maven.org/search?q=a:axon-amqp-spring-boot-starter)        |
| Axon Kafka                            | axon-kafka                           | org.axonframework.extensions.kafka       | [available](https://search.maven.org/search?q=a:axon-kafka)                           |
| Axon Kafka Spring Boot Starter        | axon-kafka-spring-boot-starter       | org.axonframework.extensions.kafka       | [available](https://search.maven.org/search?q=a:axon-kafka-spring-boot-starter)       |
| Axon Spring Cloud                     | axon-springcloud                     | org.axonframework.extensions.springcloud | [available](https://search.maven.org/search?q=a:axon-springcloud)                     |
| Axon Spring Cloud Spring Boot Starter | axon-springcloud-spring-boot-starter | org.axonframework.extensions.springcloud | [available](https://search.maven.org/search?q=a:axon-springcloud-spring-boot-starter) |
| Axon JGroups                          | axon-jgroups                         | org.axonframework.extensions.jgroups     | [available](https://search.maven.org/search?q=a:axon-jgroups)                         |
| Axon JGroups Spring Boot Starter      | axon-jgroups-spring-boot-starter     | org.axonframework.extensions.jgroups     | [available](https://search.maven.org/search?q=a:axon-jgroups-spring-boot-starter)     |
| Axon Mongo                            | axon-mongo                           | org.axonframework.extensions.mongo       | [available](https://search.maven.org/search?q=a:axon-mongo)                           |
