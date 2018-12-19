# Maven/Gradle dependencies

Axon Framework consists of a number of modules that target specific problem areas. Depending on the exact needs of your project, you will need to include one or more of these modules.

There are currently two ways of obtaining the module binaries: either download the binaries from our website or preferably configure a repository for your build system ([Maven](http://maven.apache.org/), [Gradle](https://gradle.org/)).

Axon modules are available on [Maven Central](https://search.maven.org/search?q=axonframework), so you don't have to explicitly define Axon repositories in the `pom.xml` file.

Simply, include dependencies you need, eg:
```
<dependencies>
    ...
    <dependency>
        <groupId>org.axonframework</groupId>
        <artifactId>axon-test</artifactId>
        <version>${axon.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.axonframework</groupId>
        <artifactId>axon-spring-boot-starter</artifactId>
        <version>${axon.version}</version>
    </dependency>
    ...
</dependencies>
```

## Main modules

Axon main modules are the modules that have been thoroughly tested and are robust enough to use in demanding production environments. The maven `groupId` of all these modules is `org.axonframework`. Visit [Maven Central Repository](https://search.maven.org/search?q=g:org.axonframework) to copy maven coordinates for the version you neeed to your POM file.

## Extension modules

Besides main modules, there are several extension modules which complement Axon Framework. They address distribution concerns of Axon Framework towards non-Axon Server solutions. The maven `groupId` of all these extensions starts with `org.axonframework.extensions.*`. Visit [Maven Central Repository](https://search.maven.org/search?q=axonframework%20extensions) to copy maven coordinates for the version you neeed to your POM file.