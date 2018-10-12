# Getting started

## Prerequisites
A Java 8+ JRE should be installed on the system.

## Installation procedure

The Axon Server delivery contains the following artifacts:

- axonserver.jar
- axonserver-migration.jar
- axonserver.properties
- {Cli}

Copy `axonserver.jar` and `axonserver.properties` to a directory of your choice. As Axon Server also acts
as an event store, you need to tell it where to store the events. In the provided `axonserver.properties` file
this is set up to be the events subdirectory in the startup directory. If you want to change this you can
change the value of the property `axoniq.axonserver.event.storage` to the location where you want to store the
files.

Default ports used by Axon Server are:

- 8024: HTTP port for REST calls and HTML pages
- 8124: gRPC port, used by applications to connect to Axon Server.
- 8224: Internal gRPC port, used by Axon Server nodes for internal communication

Basically, you are ready to go now. Start the Axon server using the following command:

```sh
# ./axonserver.jar
```

or when not running bash shell:

```sh
# java -jar axonserver.jar
```

To verify that the server is started correctly, open the page http://localhost:8024.