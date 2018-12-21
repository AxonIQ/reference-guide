# Clustering & Contexts

 > Note - This feature is only available on the Enterprise Edition of Axon Server

Axon Server Enterprise Edition has the concept of contexts. In a single Axon Server cluster you can support multiple bounded contexts, where events are stored in its own set of files and 
commands and queries are only routed within the bounded context. Each context is assigned to one or more nodes in the Axon Server cluster, a node can have more than one contexts.

Within a context there is one member that is leader. When storing events, replication of the data is managed by the leader. For the client this is transparent as Axon Server will 
automatically redirect the requests to the leader. Routing of commands and queries are done on all members of the context, no special role for the leader in this.  

There is one special context in Axon Server, called __admin_, this is the context that handles all administration for the complete Axon Server cluster. All requests to change contexts, define application
authorizations and users need to be sent to a node in the __admin_ context.        

You can set the following properties in the `axonserver.properties` configuration file:

* `axoniq.axonserver.name` - logical name of the node in the cluster. This must be unique within the cluster. If not set, it defaults to the hostname.
* `axoniq.axonserver.hostname` - sets the hostname as this node advertises it to clients
* `axoniq.axonserver.domain` - sets the domain as used in returning the server address to clients
* `axoniq.axonserver.internal-hostname` - hostname to be used by other nodes in the server cluster
* `axoniq.axonserver.internal-domain` - domain to be used by other nodes in the server cluster
* `axoniq.axonserver.internal-port` - internal gRPC port number for communication to other nodes

When there is no hostname specified it defaults to the hostname as returned by the _hostname_ command. 

Sample configuration:

```bash
axoniq.axonserver.hostname=axonserver1.axoniq.io
axoniq.axonserver.name=axonserver1
```

To bootstrap the Axon Server cluster initialize the cluster on the first Axon Server node (Axon Server needs to be running):

```text
# java -jar axonserver-cli.jar init-cluster -S http://node-to-add:port 
```

This creates the _admin context and a default context. 

 > This needs to be done always for Axon Server Enterprise Edition, even if you are not using clustering.  


Next for each subsequent node, run the following command to add the node to the cluster:

```text
# java -jar axonserver-cli.jar register-node -S http://node-to-add:http-port -h node-in-cluster -p internal-port-of-node-in-cluster
```

When you have a default setup with all nodes using the default port you can omit a number of parameters from this request. To connect node2 to with node1 run the following command on node2:

```text
# java -jar axoniq-cli.jar register-node -h node1
```

Default value for `-S` option is [http://localhost:8024](http://localhost:8024) and for -p is 8224 \(default internal communication port\).

This only has to be done once, each node maintains a list of all nodes in the cluster.


