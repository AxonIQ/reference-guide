# Axon Server Clustering

> Note - This feature is only available in Axon Server Enterprise

Axon Server Enterprise can be deployed as a cluster to guarantee high availability. Client applications will dynamically connect to a node in the cluster and automatically reconnect to another, should the node that they are currently connected to become unreachable.‌

Within a single cluster you can define contexts. Contexts are comparable to logical databases in a RDBMS. They allow for strong segregation without requiring deploying and managing full instances. They may be used for "bounded contexts" in the DDD sense, multi-tenancy \(with a context per tenant\), and different retention policies. When you define a context, you assign nodes that will serve that context. Data is replicated between these nodes and client connect to these nodes. For replication purposes there is one leader per context. The leader orchestrates the replication of data and confirms to clients when transactions are committed. To have a valid leader for a context, a majority of the nodes must be active \(when you have a cluster with 3 nodes, you need at least 2 active nodes, for a cluster of 4 nodes you would need 3 active nodes\).‌

Axon Server Enterprise has one special context, called "\_admin". This context is used to process all configuration changes in Axon Server, so it contains the master configuration from which all contexts get their information.‌

To get started with Axon Server Enterprise you have perform the following steps:‌

1. setup axonserver.properties files for each instance
2. start all Axon Server instances
3. initiate the cluster on one node using the "init-cluster" command line command.
4. add the other nodes to the cluster using the "register-node" command.

‌

When you start a new Axon Server Enterprise instance it does not have any contexts defined and is not capable of handling any commands from clients. This differs from starting a standard Axon Server which is immediately available for use.‌

The init-cluster commands creates 2 contexts \("\_admin" and "default"\). The default context is the context used by clients when they have not specified any context information. As this node is the only node in the cluster at this point it will be leader in both contexts.‌

Next you can add more nodes to the cluster. For these nodes, you do not call the init-cluster command, you must add them to the cluster using the register-node command. When you register a node in the cluster it will be added to all contexts by default. If you want to add it to specific contexts you can specify this in the command. You can always assign or unassign nodes from a context later, but when you have a large event store synchronizing the data to the new member may take some time.‌

All communication between Axon Server nodes uses a dedicated port \(8224 by default\). When nodes are in different locations you have to make sure that this ports is accessible for all Axon Server nodes.‌

## Properties <a id="properties"></a>

‌

A number of properties are important when setting up a cluster. Default values exist for each of the properties, but depending on your network infrastructure you may want to change them.‌

### axoniq.axonserver.name <a id="axoniq-axonserver-name"></a>

‌

The node name of the node in the Axon Server cluster. Defaults to the hostname of the machine, but if you want to test with multiple instances of Axon Server on the same machine, or you want to assign more intuitive names you can update this.‌

This must be a unique name in the cluster.‌

### axoniq.axonserver.hostname <a id="axoniq-axonserver-hostname"></a>

‌

The hostname of the node as it would be used by clients. This defaults to the hostname of the machine, however this may not be the name as it is known externally. The final hostname that is communicated to the clients is the combination of this hostname field and the axoniq.axonserver.domain property \(if set\)‌

### axoniq.axonserver.domain <a id="axoniq-axonserver-domain"></a>

‌

The domain that is added to the hostname when returning hostnames to client applications. Defaults to none.‌

### axoniq.axonserver.internal-hostname <a id="axoniq-axonserver-internal-hostname"></a>

‌

The hostname as it will be used for communication between axonserver nodes. Defaults to the axoniq.axonserver.hostname. May be used in combination with the internal-domain to make it a fully qualified name.‌

### axoniq.axonserver.internal-domain <a id="axoniq-axonserver-internal-domain"></a>

‌

Domain that is added to the internal hostnames. Defaults to axoniq.axonserver.domain.‌

### axoniq.axonserver.port <a id="axoniq-axonserver-port"></a>

‌

Port used for gRPC communication from client to Axon Server node. Defaults to 8124, when you would want to run multiple nodes on the same machine, each needs to have a unique port.‌

### axoniq.axonserver.internal-port <a id="axoniq-axonserver-internal-port"></a>

‌

Port used for gRPC communication between Axon Server nodes. Defaults to 8224, when you would want to run multiple nodes on the same machine, each needs to have a unique port.‌

### server.port <a id="server-port"></a>

‌

HTTP port used to access the UI and REST services. Defaults to 8024.‌

## Starting a cluster <a id="starting-a-cluster"></a>

‌

Once you have the properties set-up for all instances and the Axon Server nodes started, you can create the cluster, using the command line interface.‌

First step is call the init-cluster command on the first node:

```text
$ java -jar axonserver-cli.jar init-cluster -S http://[node]:[port]
```

‌

With \[node\] being the hostname of the first node and \[port\] the HTTP port. When you run this command from the first node and you are using default ports, you can omit the -S option.‌

Once you have run this command and look at the Axon Dashboard \(at [http://\[node\]:\[port](http://[node]:[port/)\]\), you will see something like this in the overview page:​![](https://gblobscdn.gitbook.com/assets%2F-M2EZHCQjtTN2AhEo25Y%2F-M2E_JZXiOKr0S0qqtaq%2F-M2E_KuV_84EcxT5F29f%2Faxonserver-overview1.png?generation=1584030678004616&alt=media)‌

The next step is to add the other nodes to the cluster, using the register-node command:

```text
$ java -jar axonserver-cli.jar register-node -S http://[node]:[port] -h [first-node] -p [internal-grpc-port]
```

‌

The values for \[node\] and \[port\] are the hostname and HTTP port for the new node to add. The \[first-node\] is the internal hostname of the first node, this should be the name that the new node uses to contact the first node. The \[internal-grpc-port\] is the port number of for internal communication on the first node, usually 8224.‌

When you run this command on the new node and you are using default ports you can omit the -S option and the -p option.‌

After adding a second node to the cluster you will see this information in the overview page:​![](https://gblobscdn.gitbook.com/assets%2F-M2EZHCQjtTN2AhEo25Y%2F-M2E_JZXiOKr0S0qqtaq%2F-M2E_KuaC7rvHHvpj_JW%2Faxonserver-overview2.png?generation=1584030678203027&alt=media)‌

You can repeat this for the third node, and then you will see the following information:​![](https://gblobscdn.gitbook.com/assets%2F-M2EZHCQjtTN2AhEo25Y%2F-M2E_JZXiOKr0S0qqtaq%2F-M2E_KucgedblYqGFl1S%2Faxonserver-overview3.png?generation=1584030677842887&alt=media)‌

## Automatic initialization <a id="automatic-initialization"></a>

> New in Axon Server 4.3

‌

You can bypass the manual configuration of the cluster by adding two additional properties in the axonserver.properties file:

```text
axoniq.axonserver.autocluster.first=internal-hostname:internal-portaxoniq.axonserver.autocluster.contexts=context1,context2
```

‌

The _axoniq.axonserver.autocluster.first_ property defines the first node in the cluster, by specifying its internal hostname \(the hostname used by other Axon Server nodes to connect to this host\), and the internal port. If the internal port is default \(8224\) it can be omitted.‌

_axoniq.axonserver.autocluster.contexts_ defines the contexts to create on the first node and to join for the other nodes. All of these contexts will be joined as primary nodes. When you don't specify any contexts, the initial node will only create an admin context, the other nodes will join the cluster, but not be a member of any contexts.‌

The autocluster properties will only take effect on a clean start of a node. If a node is already initialized, it will not create any contexts anymore, nor join the cluster again.‌

## Access control <a id="access-control"></a>

‌

Axon Server nodes expect a common token on internal requests when access control is enabled. This token must be defined in the axonserver.properties file with the property _axoniq.axonserver.accesscontrol.internal-token_. The value for this property needs to be the same on all nodes.‌

## Transport Layer Security <a id="transport-layer-security"></a>

‌

Axon Server uses the same key for the internal and external gRPC port for TLS. However you can use different certificates for the two ports. By default, Axon Server uses the value from _axoniq.axonserver.ssl.cert-chain-file_ property, but if you want to use another certificate for the internal communication set this in the property _axoniq.axonserver.ssl.internal-cert-chain-file_.‌

If you are using self-signed certificates, or certificates where the CA is not included in the cacerts keystore on the JVM, specify the CA certificate in the property _axoniq.axonserver.ssl.internal-trust-manager-file_.‌

## Internal APIs and stability guarantees <a id="internal-apis-and-stability-guarantees"></a>

‌

Within the endpoints documented in /swagger-ui.html, you can find some /internal/\* APIs that are not part of the public APIs. These APIs are not supposed to be used and will change without notice.

