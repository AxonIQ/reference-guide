# Axon Server Clustering

 > NOTE: This feature is only available on the Enterprise Edition of AxonServer

When running Axon Server in a licensed edition, you can set up a cluster of Axon servers. The servers run in active/active mode, so each node can receive and handle requests.

You can set the following properties in the `axonserver.properties` configuration file:

* `axoniq.axonserver.name` - logical name of the node in the cluster. This must be unique within the cluster.
* `axoniq.axonserver.hostname` - sets the hostname as this node advertises it to clients
* `axoniq.axonserver.domain` - sets the domain as used in returning the server address to clients
* `axoniq.axonserver.internal-hostname` - hostname to be used by other nodes in the server cluster
* `axoniq.axonserver.internal-domain` - domain to be used by other nodes in the server cluster
* `axoniq.axonserver.internal-port` - internal gRPC port number for communication to other nodes

When there is no hostname specified it defaults to the hostname as returned by the _hostname_ command.

Sample configuration:

```bash
axoniq.axonserver.hostname=messaging1.axoniq.io
axoniq.axonserver.name=messaging1
```

Connecting the nodes of a cluster is done using the command line interface. Send the register-node command to one node, specifying the address of another node in the cluster, e.g.

```text
# java -jar axoniq-cli.jar register-node -S http://node-to-add:port -h node-in-cluster -p internal-port-of-node-in-cluster
```

When you have a default setup with all nodes using the default port you can omit a number of parameters from this request. To connect node2 to with node1 run the following command on node2:

```text
# java -jar axoniq-cli.jar register-node -h node1
```

Default value for `-S` option is [http://localhost:8024](http://localhost:8024) and for -p is 8224 \(default internal communication port\).

This only has to be done once, each node maintains a list of all nodes in the cluster.

##Internal APIs and stability guarantees

Within the endpoints documented in /swagger-ui.html, you can find some /internal/* APIs that are not part of the public APIs. 
These APIs are not supposed to be used and will change without notice.

