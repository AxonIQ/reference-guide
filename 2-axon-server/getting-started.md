# Getting started

## Prerequisites

A Java 8+ JRE should be installed on the system.

## Running Axon Server locally

The Axon Server ZIP download contains executable JAR files for the server itself and the CLI. Copy `axonserver.jar` to a directory of your choice. Because Axon Server uses sensible defaults, you are now ready to go. Start the Axon Server using the following command:

```bash
$ ./axonserver.jar
```

or when not running bash shell:

```bash
$ java -jar axonserver.jar
```

When you see a log line announcing "Started Axon Server in _some-value_ seconds \(JVM running for _some-other-value_\)", the server is ready for action. To verify that the server is started correctly, open the page [http://localhost:8024](http://localhost:8024).

## Running Axon Server in a Docker container

To run Axon Server in Docker you can use the image provided on Docker Hub:

```bash
$ docker run -d --name my-axon-server -p 8024:8024 -p 8124:8124 axoniq/axonserver
```

## Other ways of running AxonServer

For more advanced options, such as running AxonServer on Kubernetes, visit [Starting AxonServer](../2.1-setup/starting-axonserver.md).
