# Maven/Gradle dependencies

Axon Framework consists of a number of modules that provide specific capabilities. 
Depending on the exact needs of your project, you will need to include one or more of these modules.

There are currently two ways of obtaining the module binaries:
 either download the binaries from our website 
 or preferably configure a repository for your build system 
 ([Maven](http://maven.apache.org/), [Gradle](https://gradle.org/)).

Axon modules are available on [Maven Central](https://search.maven.org/search?q=axonframework).

## Main modules

Axon 'Main Modules' are the modules that have been thoroughly tested and are robust enough to use in demanding
 production environments. 
The maven `groupId` of all these modules is `org.axonframework`. 
Visit [Maven Central Repository](https://search.maven.org/search?q=g:org.axonframework) to copy coordinates for the
 version you need.

> **Note**
>
> The [Axon Spring Boot Starter](maven-dependencies.md#axon-spring-boot-starter) module is the quickest start in to an
>  Axon project as it will retrieve all the required modules/dependencies transitively.
> Alternatively, you can manually select individual modules for a customized configuration.

| Module                                                                     | Artifact Id              | Group Id          | Maven Central                                                             |
| -------------------------------------------------------------------------- | ------------------------ | ----------------- |:-------------------------------------------------------------------------:|
| [Axon Messaging](maven-dependencies.md#axon-messaging)                     | axon-messaging           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-messaging)           |
| [Axon Modeling](maven-dependencies.md#axon-modeling)                       | axon-modelling           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-modelling)           |
| [Axon Event Sourcing](maven-dependencies.md#axon-event sourcing)           | axon-eventsourcing       | org.axonframework | [available](https://search.maven.org/search?q=a:axon-eventsourcing)       |
| [Axon Configuration](maven-dependencies.md#axon-configuration)             | axon-configuration       | org.axonframework | [available](https://search.maven.org/search?q=a:axon-configuration)       |
| [Axon Test](maven-dependencies.md#axon-test)                               | axon-test                | org.axonframework | [available](https://search.maven.org/search?q=a:axon-test)                |
| [Axon Server Connector](maven-dependencies.md#axon-server-connector)       | axon-server-connector    | org.axonframework | [available](https://search.maven.org/search?q=a:axon-server-connector)    |
| [Axon Spring](maven-dependencies.md#axon-spring)                           | axon-spring              | org.axonframework | [available](https://search.maven.org/search?q=a:axon-spring)              |
| [Axon Spring Boot Starter](maven-dependencies.md#axon-spring-boot-starter) | axon-spring-boot-starter | org.axonframework | [available](https://search.maven.org/search?q=a:axon-spring-boot-starter) |
| [Axon Disruptor](maven-dependencies.md#axon-disruptor)                     | axon-disruptor           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-disruptor)           |
| [Axon Metrics](maven-dependencies.md#axon-metrics)                         | axon-metrics             | org.axonframework | [available](https://search.maven.org/search?q=a:axon-metrics)             |
| [Axon Legacy](maven-dependencies.md#axon-legacy)                           | axon-legacy              | org.axonframework | [available](https://search.maven.org/search?q=a:axon-legacy)              |

### Axon Messaging 
This module contains all necessary components and building blocks to support command, event and query messaging.

### Axon Modeling
This module contains the necessary components to create domain models, like Aggregates and Sagas.

### Axon Event Sourcing 
This module contains all necessary infrastructure components to support Event Sourcing, Command and Query Models.

### Axon Test
This module contains test fixtures that you can use to test Axon based components,
 such as your Command Handlers, Aggregates and Sagas. 
You typically do not need this module at runtime and will only need to be added to the classpath for running tests.

### Axon Configuration
This module contains all the necessary components to configure an Axon application.

### Axon Server Connector 
This module provides infrastructure components that connect to Axon Server.

### Axon Spring 
This module allows Axon Framework components to be configured in the Spring Application context. 
It also provides a number of building block implementations specific to Spring Framework,
 such as an adapter for publishing and retrieving Axon Events on a Spring Messaging Channel.

### Axon Spring Boot Starter
This module provides Spring Boot auto-configuration for your project. 
It is by far the easiest option to get started as it automatically configures all Axon components. 
It is explained in more details [here](spring-boot.md).

### Axon Disruptor
This module contains a specific CommandBus and Command Handling solution based on the Disruptor paradigm.

### Axon Metrics
This module provides basic implementations based on Coda Hale to collect the monitoring information.

### Axon Legacy
This module contains components that enable migration of older Axon projects to use the latest Axon version.

## Extension modules

Besides main modules, there are several extension modules which complement Axon Framework. 
They address distribution concerns of Axon Framework towards non-Axon Server solutions. 
The maven `groupId` of these extensions starts with `org.axonframework.extensions.*`. 
Visit [Maven Central Repository](https://search.maven.org/search?q=axonframework%20extensions) to copy coordinates for the version you need.

| Module                                                                                          | Artifact Id                          | Group Id                                 | Maven Central                                                                         |
| ----------------------------------------------------------------------------------------------- | ------------------------------------ | ---------------------------------------- |:-------------------------------------------------------------------------------------:|
| [Axon AMQP](maven-dependencies.md#axon-amqp)                                                    | axon-amqp                            | org.axonframework.extensions.amqp        | [available](https://search.maven.org/search?q=a:axon-amqp)                            |
| [Axon AMQP Spring Boot Starter](maven-dependencies.md#axon-amqp-spring-boot-starter)            | axon-amqp-spring-boot-starter        | org.axonframework.extensions.amqp        | [available](https://search.maven.org/search?q=a:axon-amqp-spring-boot-starter)        |
| [Axon Kafka](maven-dependencies.md#axon-kafka)                                                  | axon-kafka                           | org.axonframework.extensions.kafka       | [available](https://search.maven.org/search?q=a:axon-kafka)                           |
| [Axon Kafka Spring Boot Starter](maven-dependencies.md#axon-kafka-spring-boot-starter)          | axon-kafka-spring-boot-starter       | org.axonframework.extensions.kafka       | [available](https://search.maven.org/search?q=a:axon-kafka-spring-boot-starter)       |
| [Axon Spring Cloud](maven-dependencies.md#axon-spring cloud)                                    | axon-springcloud                     | org.axonframework.extensions.springcloud | [available](https://search.maven.org/search?q=a:axon-springcloud)                     |
| [Axon Spring Cloud Spring Boot Starter](maven-dependencies.md#axon-jgroups-spring-boot-starter) | axon-springcloud-spring-boot-starter | org.axonframework.extensions.springcloud | [available](https://search.maven.org/search?q=a:axon-springcloud-spring-boot-starter) |
| [Axon JGroups](maven-dependencies.md#axon-jgroups)                                              | axon-jgroups                         | org.axonframework.extensions.jgroups     | [available](https://search.maven.org/search?q=a:axon-jgroups)                         |
| [Axon JGroups Spring Boot Starter](maven-dependencies.md#axon-jgroups-spring-boot-starter)      | axon-jgroups-spring-boot-starter     | org.axonframework.extensions.jgroups     | [available](https://search.maven.org/search?q=a:axon-jgroups-spring-boot-starter)     |
| [Axon Mongo](maven-dependencies.md#axon-mongo)                                                  | axon-mongo                           | org.axonframework.extensions.mongo       | [available](https://search.maven.org/search?q=a:axon-mongo)                           |
| [Axon CDI](maven-dependencies.md#axon-cdi)                                                      | axon-cdi                             | org.axonframework.extensions.cdi         | [available](https://search.maven.org/search?q=a:axon-cdi)                             |


### Axon AMQP
This module provides components that allow you leverage an AMQP-based message broker as an Event Message distribution
 mechanism. 
This allows for guaranteed delivery, even when the Event Handler node is temporarily unavailable.

### Axon AMQP Spring Boot Starter
This module provides Spring auto-configuration on top of the `axon-amqp` module.

### Axon Kafka
This module provides integration with Kafka for event distribution.
As such it plays a similar role as the [Axon AMQP](maven-dependencies.md#axon-amqp) extension
 and thus is **not** a replacement Event Storage mechanism.
[Kafka](https://kafka.apache.org/) is a distributed message streaming platform.

### Axon Kafka Spring Boot Starter
This module provides Spring auto-configuration on top of the `axon-kafka` module.

### Axon Spring Cloud
This module provides integration with Spring Cloud for command distribution.
[Spring Cloud](https://spring.io/projects/spring-cloud) provides an API for common distributed system patterns.

### Axon Spring Cloud Spring Boot Starter
This module provides Spring auto-configuration on top of the `axon-springcloud` module

### Axon JGroups 
This module provides integration with JGroups for command distribution.
[JGroups](http://www.jgroups.org/) should be regarded as a reliable messaging toolkit. 

### Axon JGroups Spring Boot Starter
This module provides Spring auto-configuration on top of the `axon-jgroups` module

### Axon Mongo
This module provides event and saga store implementations that store event streams and sagas in a MongoDB database. 
[MongoDB](https://www.mongodb.com/) is a document based NoSQL database. 

### Axon CDI
This module provides support for Contexts and Dependency Injection (CDI) for the Java EE platform.