# Starting the Axon Server

There is more then one way to start and run Axon Server:

## Starting Axon Server locally

The [Axon download page](https://axoniq.io/download) contains executable JAR files for the server itself and the CLI. Copy `axonserver.jar` to a directory of your choice. Because Axon Server uses sensible defaults, you are now ready to go. Start the Axon Server using the following command:

```bash
$ ./axonserver.jar
```

or when not running bash shell:

```bash
$ java -jar axonserver.jar
```

When you see a log line announcing "Started Axon Server in _some-value_ seconds \(JVM running for _some-other-value_\)", the server is ready for action. To verify that the server is started correctly, open the page [http://localhost:8024](http://localhost:8024).

## Alternative ways of running the Axon Server

Axon provides a Docker image of Axon Server, which is [available on DockerHub](https://hub.docker.com/r/axoniq/axonserver/).

Alternatively, you can use this image to run Axon Server in Docker container and deploy it to Kubernetes.

See `Operations Guide` chapter \(section: [Setting up Axon Server/Launch]()\) for more details.

