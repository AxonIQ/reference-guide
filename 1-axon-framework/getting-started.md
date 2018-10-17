# Getting started

This section will explain how you can obtain the binaries for Axon to get started. There are currently two ways: either download the binaries from our [website](https://axoniq.io/product-overview/axon-framework) or configure a repository for your build system \([Maven](https://mvnrepository.com/artifact/org.axonframework/axon), Gradle, etc\).

## Download Axon

You can download the Axon Framework from our downloads page: [https://axoniq.io/product-overview/axon-framework](https://axoniq.io/product-overview/axon-framework)

If you really want to stay on the bleeding edge of development, you can clone the Git repository: [git://github.com/AxonFramework/AxonFramework.git](git://github.com/AxonFramework/AxonFramework.git), or visit [https://github.com/AxonFramework/AxonFramework](https://github.com/AxonFramework/AxonFramework) to browse the sources online.

## Configure Maven

If you use Maven as your build tool, you need to configure the correct dependencies for your project. Add the following code in your dependencies section:

```markup
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-core</artifactId>
    <version>${axon.version}</version>
</dependency>
```

Most of the features provided by the Axon Framework are optional and require additional dependencies. We have chosen not to add these dependencies by default, as they would potentially clutter your project with artifacts you would not require.

## Infrastructure requirements

Axon Framework does not impose many requirements on the infrastructure. It has been built and tested against Java 8, making that more or less the only requirement.

Since Axon doesn't create any connections or threads by itself, it is safe to run on an Application Server. Axon abstracts all asynchronous behavior by using `Executor`s, meaning that you can easily pass a container managed Thread Pool, for example. If you don't use a full blown Application Server \(e.g. Tomcat, Jetty or a stand-alone app\), you can use the `Executors` class or the Spring Framework to create and configure Thread Pools.

## When you are stuck

While implementing your application, you might run into problems, wonder about why certain things are the way they are, or have some questions that need an answer. The Axon Users [mailing](https://groups.google.com/forum/#!forum/axonframework) list is there to help. Just send an email to [axonframework@googlegroups.com](mailto:axonframework@googlegroups.com). Other users as well as contributors to the Axon Framework are there to help with your issues.

If you find a bug, you can report them at our [GitHub](https://github.com/AxonFramework/AxonFramework/issues) issues page. When reporting an issue, please make sure you clearly describe the problem. Explain what you did, what the result was and what you expected to happen instead. If possible, please provide a very simple Unit Test \(JUnit\) that shows the problem. That makes fixing it a lot simpler.

