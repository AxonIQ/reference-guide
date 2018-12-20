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

### Axon Test
This module contains test fixtures that you can use to test Axon based components, such as your Command Handlers, Aggregates and Sagas. You typically do not need this module at runtime and will only need to be added to the classpath for running tests.

### Axon Server Connector 
This module provides infrastructure components that connect to AxonServer.

### Axon Configuration
This module contains all the necessary components to configure an Axon application.

### Axon Spring 
This module allows Axon Framework components to be configured in the Spring Application context. It also provides a number of building block implementations specific to Spring Framework, such as an adapter for publishing and retrieving Axon Events on a Spring Messaging Channel.

### Axon Spring Boot Starter
This module provides Spring Boot starters on top of the `axon-spring` module, so you can benefit from Spring auto-configuration.

### Axon Messaging 
This module contains all necessary components and building blocks to support command, event and query messaging.

### Axon Modeling
This module contains the necessary components to create domain models, like Aggregates and Sagas.

### Axon Eventsourcing 
This module contains all necessary infrastructure components to support Event Sourcing Command and Query models.

### Axon Disruptor
This module contains a specific CommandBus and Command Handling solution based on the Disruptor paradigm.

### Axon Metrics
This module provides basic implementations based on Coda Hale to collect the monitoring information.

### Axon CDI
This module provides support for Contexts and Dependency Injection (CDI) for the Java EE platform.

### Axon Legacy
This module contains components that enable migration of older Axon projects to use the latest Axon version.

## Extension modules

Besides main modules, there are several extension modules which complement Axon Framework. They address distribution concerns of Axon Framework towards non-Axon Server solutions. The maven `groupId` of all these extensions starts with `org.axonframework.extensions.*`. Visit [Maven Central Repository](https://search.maven.org/search?q=axonframework%20extensions) to copy coordinates for the version you neeed.

| Module                                                                     | Artifact Id                          | Group Id                                 | Maven Central                                                                         |
| -------------------------------------------------------------------------- | ------------------------------------ | ---------------------------------------- |:-------------------------------------------------------------------------------------:|
| [Axon AMQP](#axon-amqp )                                                   | axon-amqp                            | org.axonframework.extensions.amqp        | [available](https://search.maven.org/search?q=a:axon-amqp)                            |
| [Axon AMQP Spring Boot Starter](#axon-amqp-spring-boot-starter)            | axon-amqp-spring-boot-starter        | org.axonframework.extensions.amqp        | [available](https://search.maven.org/search?q=a:axon-amqp-spring-boot-starter)        |
| [Axon Kafka](#axon-kafka)                                                  | axon-kafka                           | org.axonframework.extensions.kafka       | [available](https://search.maven.org/search?q=a:axon-kafka)                           |
| [Axon Kafka Spring Boot Starter](#axon-kafka-spring-boot-starter)          | axon-kafka-spring-boot-starter       | org.axonframework.extensions.kafka       | [available](https://search.maven.org/search?q=a:axon-kafka-spring-boot-starter)       |
| [Axon Spring Cloud](#axon-springcloud)                                     | axon-springcloud                     | org.axonframework.extensions.springcloud | [available](https://search.maven.org/search?q=a:axon-springcloud)                     |
| [Axon Spring Cloud Spring Boot Starter](#axon-jgroups-spring-boot-starter) | axon-springcloud-spring-boot-starter | org.axonframework.extensions.springcloud | [available](https://search.maven.org/search?q=a:axon-springcloud-spring-boot-starter) |
| [Axon JGroups](#axon-jgroups)                                              | axon-jgroups                         | org.axonframework.extensions.jgroups     | [available](https://search.maven.org/search?q=a:axon-jgroups)                         |
| [Axon JGroups Spring Boot Starter](#axon-jgroups-spring-boot-starter)      | axon-jgroups-spring-boot-starter     | org.axonframework.extensions.jgroups     | [available](https://search.maven.org/search?q=a:axon-jgroups-spring-boot-starter)     |
| [Axon Mongo](#axon-mongo)                                                   | axon-mongo                           | org.axonframework.extensions.mongo       | [available](https://search.maven.org/search?q=a:axon-mongo)                           |


### Axon AMQP
This module provides components that allow you to build up an event bus using an AMQP-based message broker as distribution mechanism. This allows for guaranteed-delivery, even when the event handler node is temporarily unavailable.

### Axon AMQP Spring Boot Starter
This module provides components provides Spring auto-configuration on top of the `axon-amqp` module.

### Axon Kafka
This module provides integration with Kafka for event distribution (do note that events are not stored in this scenario).

### Axon Kafka Spring Boot Starter
This module provides components provides Spring auto-configuration on top of the `axon-kafka` module.

### Axon Spring Cloud
This module provides integration with Spring Cloud for command distribution.

### Axon Spring Cloud Spring Boot Starter
This module provides components provides Spring auto-configuration on top of the `axon-jgroups` module

### Axon JGroups 
This module provides integration with JGroups for command distribution.

### Axon JGroups Spring Boot Starter
This module provides components provides Spring auto-configuration on top of the `axon-jgroups` module

### Axon Mongo
This module provides event and saga store implementations that store event streams and sagas in a MongoDB database. [MongoDB](https://www.mongodb.com/) is a document based NoSQL database. 