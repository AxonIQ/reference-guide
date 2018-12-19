# Maven/Gradle dependencies

Axon Framework consists of a number of modules that target specific problem areas. Depending on the exact needs of your project, you will need to include one or more of these modules.

There are currently two ways of obtaining the module binaries: either download the binaries from our website or preferably configure a repository for your build system ([Maven](http://maven.apache.org/), [Gradle](https://gradle.org/)).

Axon modules are available on [Maven Central](https://search.maven.org/search?q=axonframework).

## Main modules

Axon main modules are the modules that have been thoroughly tested and are robust enough to use in demanding production environments. The maven `groupId` of all these modules is `org.axonframework`. Visit [Maven Central Repository](https://search.maven.org/search?q=g:org.axonframework) to copy coordinates for the version you neeed.

| Module                        | Artifact Id              | Group Id          | Maven Central                                                             |
| ----------------------------- | ------------------------ | ----------------- |:-------------------------------------------------------------------------:|
| Axon Test                     | axon-test                | org.axonframework | [available](https://search.maven.org/search?q=a:axon-test)                |
| Axon Server Connector         | axon-server-connector    | org.axonframework | [available](https://search.maven.org/search?q=a:axon-server-connector)    |
| Axon Configuration            | axon-configuration       | org.axonframework | [available](https://search.maven.org/search?q=a:axon-configuration)       |
| Axon Spring                   | axon-spring              | org.axonframework | [available](https://search.maven.org/search?q=a:axon-spring)              |
| Axon Spring Boot Starter      | axon-spring-boot-starter | org.axonframework | [available](https://search.maven.org/search?q=a:axon-spring-boot-starter) |
| Axon Messaging                | axon-messaging           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-messaging)           |
| Axon Modeling                 | axon-modelling           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-modelling)           |
| Axon Eventsourcing            | axon-eventsourcing       | org.axonframework | [available](https://search.maven.org/search?q=a:axon-eventsourcing)       |
| Axon Disruptor                | axon-disruptor           | org.axonframework | [available](https://search.maven.org/search?q=a:axon-disruptor)           |
| Axon Metrics                  | axon-metrics             | org.axonframework | [available](https://search.maven.org/search?q=a:axon-metrics)             |
| Axon CDI                      | axon-cdi                 | org.axonframework | [available](https://search.maven.org/search?q=a:axon-cdi)                 |
| Axon Legacy                   | axon-legacy              | org.axonframework | [available](https://search.maven.org/search?q=a:axon-legacy)              |




## Extension modules

Besides main modules, there are several extension modules which complement Axon Framework. They address distribution concerns of Axon Framework towards non-Axon Server solutions. The maven `groupId` of all these extensions starts with `org.axonframework.extensions.*`. Visit [Maven Central Repository](https://search.maven.org/search?q=axonframework%20extensions) to copy coordinates for the version you neeed.