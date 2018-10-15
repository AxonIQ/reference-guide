# 3.1 Configuration

## Names and ports

* `axoniq.axonserver.name`: the unique node name of the AxonServer node is taken from the hostname of the server.
* `axoniq.axonserver.hostname`: the hostname of the AxonServer node, as returned to clients, is taken from the hostname of the server.
* `axoniq.axonserver.port`: the gRPC port for clients to connect is set to 8124 by default
* `server.port`: the HTTP port for REST clients to connect is set to 8024 by default

## Database

By default each AxonServer node will create its own H2 database in a file axonserver-controldb in the working directory. To change this, set the property:

* `axoniq.axonserver.controldb-path`: Path to controlDB

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

## Cluster \[Not in Free edition\]

When runnning AxonServer in a licensed edition, you can set up a cluster of Axon servers. The servers run in active/active mode, so each node can receive and handle requests.

You can set the following properties in the axonserver.properties configuration file:

* `axoniq.axonserver.name` - logical name of the node in the cluster. _This must be unique within the cluster_.
* `axoniq.axonserver.hostname` - sets the hostname as this node advertises it to clients
* `axoniq.axonserver.domain` - sets the domain as used in returning the server address to clients
* `axoniq.axonserver.internal-hostname` - hostname to be used by other nodes in the server cluster
* `axoniq.axonserver.internal-domain` - domain to be used by other nodes in the server cluster
* `axoniq.axonserver.internal-port` - internal gRPC port number for communication to other nodes

When there is no hostname specified it defaults to the hostname as returned by the _hostname_ command.

Sample configuration:

```text
axoniq.axonserver.hostname=messaging1.axoniq.io
axoniq.axonserver.name=messaging1
```

Connecting the nodes of a cluster is done using the command line interface. Send the register-node command to one node, specifying the address of another node in the cluster, e.g.

```bash
# {CliCmd} register-node -S http://node-to-add:port -h node-in-cluster -p internal-port-of-node-in-cluster
```

When you have a default setup with all nodes using the default port you can omit a number of parameters from this request. To connect node2 to with node1 run the following command on node2:

```bash
# {CliCmd} register-node -h node1
```

Default value for -S option is [http://localhost:8024](http://localhost:8024) and for -p is 8224 \(default internal communication port\).

This only has to be done once, each node maintains a list of all nodes in the cluster.

## Flow control

Flow control is the process of managing the rate of data transmission between two nodes to prevent a fast sender from overwhelming a slow receiver.

In the messaging platform flow control is possible both between the messaging platform and the message handlers, and between the nodes in the messaging platform cluster.

Messaging platform - message handler::

The client needs to set the following properties to configure flow control:

* `axon.axonserver.initial-nr-of-permits` \[100000\] - number of messages that the server can initially send to client.
* `axon.axonserver.nr-of-new-permits` \[90000\] - additional number of messages that the server can send to client.
* `axon.axonserver.new-permits-threshold` \[10000\] -  when client reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

AxonServer nodes::

Set the following properties to set flow control on the synchronization between nodes in an AxonServer cluster:

* `axoniq.axonserver.commandFlowControl.initial-nr-of-permits` \[100000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.commandFlowControl.nr-of-new-permits` \[90000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.commandFlowControl.new-permits-threshold` \[10000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.
* `axoniq.axonserver.queryFlowControl.initial-nr-of-permits` \[100000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.queryFlowControl.nr-of-new-permits` \[90000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.queryFlowControl.new-permits-threshold` \[10000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

## Access control

To enable access control add the following property to the axonserver.properties:

* _axoniq.axonserver.accesscontrol.enabled_=true

To register an application and get an access token use the following command:

```text
{CliCmd} register-application -S http://messaging:8080 -a applicationname -d description -r READ,WRITE,ADMIN
```

The address of the server specified in this command is the address of the current master node. The master will distribute the applications to all the replicas.

This command returns the generated token to use. _Note that this token is only generated once, if you loose it you must delete the application and register it again to get a new token_. If you want to define the token yourself, you can provide one in the command line command using the -T flag, e.g.:

```text
{CliCmd} register-application -a applicationname -d description -r READ,WRITE -T this-is-my-token
```

The minimum length for a token is 8 characters.

Specify the access token in the client by setting the property:

* `axon.axonserver.token`=\[Token\]

In the Free Edition it is not possible to create applications. If you want to use access control in this edition specify the property `axoniq.axonserver.accesscontrol.token` with any value you want on the Axon server and set the same value in the `axoniq.axonserver.token` property on the client.

You can access the Axon webpages when access control is enabled by providing a username and password. Users are created through the command line using the following command:

\[subs="attributes"\]

```text
{CliCmd} register-user -S http://axonserver:8024 -u USERNAME -r USER,ADMIN
```

The command will prompt for a password. If you want to set the password directly you can use the -p option.

Note that the command line interface also requires a token when executing remote commands. If you execute a command from an AxonServer node itself, you do not need to provide a token.

If access control is enabled and AxonServer is running in clustered mode, the nodes in the cluster also need to attach a token with the messages sent between them. This token must be defined in the application properties file on each node with property `axoniq.axonserver.accesscontrol.internal-token`. The value of this property must be the same for all nodes.

## SSL

Requires private key and public certificate. The same certificate is used to connect to all event store servers.

For gRPC communication add file locations to the axonserver.properties file:

* `axoniq.axonserver.ssl.cert-chain-file` - location of the public certificate file
* `axoniq.axonserver.ssl.private-key-file` - location of the private key file

## Sample:

axoniq.axonserver.ssl.enabled=true axoniq.axonserver.ssl.cert-chain-file=./resources/axoniq-public.crt

## axoniq.axonserver.ssl.private-key-file=./resources/axoniq-private.pem

For HTTPs configuration the certificate and key need to be installed in a p12 keystore. To install the keys use the following command:

```text
openssl pkcs12 -export -in [public-cert-file] -inkey [private-key-file] -out [keystore-file] -name [alias]
```

## Configure the Messaging Platform server to use HTTPs by adding the following properties to the axonserver.properties file:

server.port=8443 server.ssl.key-store=\[keystore-file\]   
server.ssl.key-store-password=\[password\] server.ssl.key-store-type=PKCS12

## server.ssl.key-alias=\[alias\]

## Multi-context \[Enterprise Edition only\]

You can use a single AxonServer \(cluster\) to store events for multiple bounded contexts. Each context will have its own set of files \(stored in a separate directory\). Each context may have a different master in an AxonDB cluster.

Creating a new context is done using the command line interface:

```text
{CliCmd} register-context -S http://axonserver:8024 -c context-name
```

The server address here is the address of the master for the `default` context. It is not possible to delete the `default` context.

## Migration

The AxonServer package contains a migration tool to migrate from an already existing RDBMS event store to a new AxonServer instance. The tool reads events and snapshots from the existing store and pushes them to the AxonServer server.

The migration tool maintains the state of its migration, so it can be run multiple times.

Set the following properties to define the existing event store and the target AxonServer server:

* `axoniq.axonserver.servers` - comma separated list of hostnames and ports for the AxonServer cluster.
* `axoniq.datasource.eventstore.url` - URL of the JDBC data store containing the existing event store
* `axoniq.datasource.eventstore.username` - Username to connect to the JDBC data store containing the existing event store
* `axoniq.datasource.eventstore.password` - Password to connect to the JDBC data store containing the existing event store

The default settings expect the data in the current event store to be serialized using the XstreamSerializer. When the data is serialized using the JacksonSerializer add the following property:

* `axon.serializer.events*=jackson`

To run the migration tool create a file application.properties, containing the properties mentioned above, e.g.

```text
axoniq.axonserver.servers=localhost
axoniq.datasource.eventstore.url=jdbc:mysql://localhost:3306/applicationdb?useSSL=false
axoniq.datasource.eventstore.username=myusername
axoniq.datasource.eventstore.password=mypassword
```

Run the command `axonserver-migration.jar`

When the source eventstore is requiring a specific JDBC driver, you should put the required JDBC driver jar files in the libs directory.

Note that the migration tool only migrates the event store data to AxonServer. It does not update the tracking token values in token\_entry tables. Tracking tokens are highly dependent on the implementation of the actual event store used. Migrating them is case specific and error prone. Our recommendation is to reset the tracking processors after the migration.

