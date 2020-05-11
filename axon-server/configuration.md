# Configuration

There are several configuration options for the Axon Server that can be done to streamline and optimize your Axon Server deployment.

A list of the important ones is provided below. The configuration is maintained in the _axonserver.properties_ file which is available as part of the server distribution.

## Names and ports

* `axoniq.axonserver.name` - unique node name of the Axon Server node is taken from the hostname of the server.
* `axoniq.axonserver.hostname` - hostname of the Axon Server node, as returned to clients, is taken from the hostname of the server.
* `axoniq.axonserver.port` - gRPC port for clients to connect is set to 8124 by default
* `server.port` - HTTP port for REST clients to connect is set to 8024 by default

## Data Directory

### Events and Snapshot Events

AxonServer stores all Events and Snapshot Events in segmented files on disk. By default, these files are stored in the ./data directory.

The following settings define an alternative storage location:

* `axoniq.axonserver.event.storage` - path where \(regular\) events are stored
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

_Messaging Platform - Messaging Handler:_

The client \(i.e. the Axon Application\) needs to set the following properties to configure flow control:

* `axon.axonserver.initial-nr-of-permits` \[1000\] - number of messages that the server can initially send to client.
* `axon.axonserver.nr-of-new-permits` \[500\] - additional number of messages that the server can send to client.
* `axon.axonserver.new-permits-threshold` \[500\] -  when client reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

_Axon Server Nodes:_

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



