# Spring Ahead of Time

[Spring AOT processing](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.introducing-graalvm-native-images.understanding-aot-processing)
is part of the process to create a native binary from a Spring (Boot) application. This extension will help in adding a
lot of hints which are needed. Please note this extension can only be used with Spring Boot 3, as
such it requires at least Java 17.

Besides the extension it might be needed to make more changes to successfully compile and run an application as native
image. For example, when a message isn't use in a handler. This is quite common when the application is split, and the
application sending certain messages is not the same as the application handling the messages. In those cases these
messages need to be added to
the [ImportRuntimeHints](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ImportRuntimeHints.html)
annotation. Otherwise, these messages can't be deserialized, leading to errors at runtime.

If something is not working, or only works with additional hints, and it's Axon Specific, please let us know either
at [GitHub](https://github.com/AxonFramework/extension-spring-aot/issues)
or [Discuss](https://discuss.axoniq.io/c/axonframework/7).

## Compiling to native

It's important to read through
the [documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html) from Spring
itself. There are some known limitations which might require additional changes to the application.
In addition, this extension needs to be added by adding the dependency like:

```xml

<dependency>
    <groupId>org.axonframework.extensions.spring-aot</groupId>
    <artifactId>axon-spring-aot</artifactId>
    <version>4.8.0</version>
</dependency>
```

This should be enough to have additional hints with ahead of time compilation to successfully build and run your Axon
application.

## Performance tips

It can be beneficial to move from JPA implementations to JDBC implementations. This likely decreases both the time it
takes to compile the image and the time to start the image.


