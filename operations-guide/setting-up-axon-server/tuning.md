# Tuning

## Names and ports

* `axoniq.axonserver.name` - unique node name of the Axon Server node is taken from the hostname of the server.
* `axoniq.axonserver.hostname` - hostname of the Axon Server node, as returned to clients, is taken from the hostname of the server.
* `axoniq.axonserver.port` - gRPC port for clients to connect is set to 8124 by default
* `server.port` - HTTP port for REST clients to connect is set to 8024 by default

## Data directory

### Events and Snapshot Events

AxonServer stores all Events and Snapshot Events in segmented files on disk. By default, these files are stored in the ./data directory.

The following settings define an alternative storage location:
* `axoniq.axonserver.event.storage` - path where (regular) events are stored
* `axoniq.axonserver.snapshot.storage` - path where Snapshot Events are stored

### Control database

By default each Axon Server node will create its own H2 database in a file axonserver-controldb in the data directory. To change this, set the property:

* `axoniq.axonserver.controldb-path` - path to controlDB

## Logging

Logging is by default set to WARN level for all packages. To change the logging levels for specific packages or classes add logging level properties to the axonserver.properties file, for example:

```text
logging.level.io.axoniq.axonserver=INFO
```

Log messages are written to stdout by default. To change this add the properties logging.file and/or logging.path to the axonserver.properties file.

```bash
# write log entries to file called messaging.log
logging.file=messaging.log
# create logfiles in directory /var/log
logging.path=/var/log
```

## Flow control

Flow control is the process of managing the rate of data transmission between two nodes to prevent a fast sender from overwhelming a slow receiver.

In the messaging platform flow control is possible both between the messaging platform and the message handlers, and between the nodes in the messaging platform cluster.

Messaging platform - message handler:

The client needs to set the following properties to configure flow control:

* `axon.axonserver.initial-nr-of-permits` \[1000\] - number of messages that the server can initially send to client.
* `axon.axonserver.nr-of-new-permits` \[500\] - additional number of messages that the server can send to client.
* `axon.axonserver.new-permits-threshold` \[500\] -  when client reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

Axon Server nodes:

Set the following properties to set flow control on the synchronization between nodes in an Axon Server cluster:

* `axoniq.axonserver.commandFlowControl.initial-nr-of-permits` \[10000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.commandFlowControl.nr-of-new-permits` \[5000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.commandFlowControl.new-permits-threshold` \[5000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.
* `axoniq.axonserver.queryFlowControl.initial-nr-of-permits` \[10000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.queryFlowControl.nr-of-new-permits` \[5000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.queryFlowControl.new-permits-threshold` \[5000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

## SSL

Requires private key and public certificate. The same certificate is used to connect to all event store servers.

For gRPC communication add file locations to the `axonserver.properties` file:

* `axoniq.axonserver.ssl.cert-chain-file` - location of the public certificate file
* `axoniq.axonserver.ssl.private-key-file` - location of the private key file

Sample:

```text
axoniq.axonserver.ssl.enabled=true 
axoniq.axonserver.ssl.cert-chain-file=./resources/axoniq-public.crt
axoniq.axonserver.ssl.private-key-file=./resources/axoniq-private.pem
```

For HTTPs configuration the certificate and key need to be installed in a p12 keystore. To install the keys use the following command:

```text
$ openssl pkcs12 -export -in [public-cert-file] -inkey [private-key-file] -out [keystore-file] -name [alias]
```

Configure the Messaging Platform server to use HTTPs by adding the following properties to the axonserver.properties file:

```text
server.port=8443 
server.ssl.key-store=[keystore-file]   
server.ssl.key-store-password=[password] 
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=[alias]
```

## Migration

The Axon Server package contains a migration tool to migrate from an already existing RDBMS event store to a new Axon Server instance. The tool reads events and snapshots from the existing store and pushes them to the Axon Server server.

The migration tool maintains the state of its migration, so it can be run multiple times.

Set the following properties to define the existing event store and the target Axon Server server:

* `axoniq.axonserver.servers` - comma separated list of hostnames and ports for the Axon Server cluster.
* `axoniq.datasource.eventstore.url` - url of the JDBC data store containing the existing event store
* `axoniq.datasource.eventstore.username` - username to connect to the JDBC data store containing the existing event store
* `axoniq.datasource.eventstore.password` - password to connect to the JDBC data store containing the existing event store

The default settings expect the data in the current event store to be serialized using the `XstreamSerializer`. When the data is serialized using the JacksonSerializer add the following property:

* `axon.serializer.events*=jackson`

To run the migration tool create a file `application.properties`, containing the properties mentioned above, e.g.

```text
axoniq.axonserver.servers=localhost
axoniq.datasource.eventstore.url=jdbc:mysql://localhost:3306/applicationdb?useSSL=false
axoniq.datasource.eventstore.username=myusername
axoniq.datasource.eventstore.password=mypassword
```

Run the command `axonserver-migration.jar`

When the source event store is requiring a specific JDBC driver, you should put the required JDBC driver jar files in the libs directory.

Note that the migration tool only migrates the event store data to Axon Server. It does not update the tracking token values in token\_entry tables. Tracking tokens are highly dependent on the implementation of the actual event store used. Migrating them is case specific and error prone. Our recommendation is to reset the tracking processors after the migration.
