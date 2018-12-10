# Maven/Gradle dependencies

Axon Framework consists of a number of modules that target specific problem areas. Depending on the exact needs of your project, you will need to include one or more of these modules.

There are currently two ways of obtaining the module binaries: either download the binaries from our website or preferably configure a repository for your build system (Maven, Gradle, etc).

Axon modules are available on Maven central, so you don't have to explicitly define Axon repositories in the `pom.xml` file.

Simply, include dependencies you need, eg:
```
<dependencies>
    ...
    <!-- Axon main modules -->
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
    <!-- Axon extension modules-->
     <dependency>
        <groupId>org.axonframework.extensions.amqp</groupId>
        <artifactId>axon-amqp-spring-boot-starter</artifactId>
        <version>${axon-extension.version}</version>
    </dependency>
    ...
</dependencies>
```

## Main modules

[Axon main modules](https://mvnrepository.com/artifact/org.axonframework) are the modules that have been thoroughly tested and are robust enough to use in demanding production environments. The maven groupId of all these modules is `org.axonframework`.

## Extension modules

Besides main modules, there are several extension modules which complement Axon Framework. They address distribution concerns of Axon Framework towards non-Axon Server solutions. The maven groupId of all these extensions is `org.axonframework.extensions`.