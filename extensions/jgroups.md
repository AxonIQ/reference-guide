# JGroups

JGroups is an alternative approach to distributing command bus \(commands\), besides Axon Server which is the default.

The `JGroupsConnector` uses \(as the name already gives away\) JGroups as the underlying discovery and dispatching mechanism. Describing the feature set of JGroups is a bit too much for this reference guide, so please refer to the [JGroups User Guide](http://www.jgroups.org/ug.html) for more details.

To use the Spring JGroups components from Axon, make sure the `axon-jgroups` module is available on the classpath.

Since JGroups handles both discovery of nodes and the communication between them, the `JGroupsConnector` acts as both a `CommandBusConnector` and a `CommandRouter`.

> **Note**
>
> You can find the JGroups specific components for the `DistributedCommandBus` in the `axon-distributed-commandbus-jgroups` module.

The `JGroupsConnector` has four mandatory configuration elements:

* `channel` - which defines the JGroups protocol stack. Generally, a JChannel is constructed with a reference to a JGroups configuration file. JGroups comes with a number of default configurations which can be used as a basis for your own configuration. Do keep in mind that IP Multicast generally doesn't work in Cloud Services, like Amazon. TCP Gossip is generally a good start in such type of environment.
* `clusterName` - defines the name of the cluster that each segment should register to. Segments with the same cluster name will eventually detect each other and dispatch commands among each other.
* `localSegment` - the Command Bus implementation that dispatches Commands destined for the local JVM. These commands may have been dispatched by instances on other JVMs or from the local one.
* `serializer`- used to serialize command messages before they are sent over the wire.

> **Note**
>
> When using a cache, it should be cleared out when the `ConsistentHash` changes to avoid potential data corruption \(e.g. when commands do not specify a `@TargetAggregateVersion` and a new member quickly joins and leaves the JGroup, modifying the aggregate while it is still cached elsewhere.\)

Ultimately, the `JGroupsConnector` needs to actually connect in order to dispatch messages to other segments. To do so, call the `connect()` method.

```java
JChannel channel = new JChannel("path/to/channel/config.xml");
CommandBus localSegment = SimpleCommandBus.builder().build();
Serializer serializer = XStreamSerializer.builder().build();

JGroupsConnector connector = JGroupsConnector.builder()
                                             .channel(channel)
                                             .clusterName("myCommandBus")
                                             .localSegment(localSegment)
                                             .serializer(serializer)
                                             .build();
DistributedCommandBus commandBus = DistributedCommandBus.builder()
                                                        .connector(connector)
                                                        .commandRouter(connector)
                                                        .build();

// on one node:
commandBus.subscribe(CommandType.class.getName(), handler);
connector.connect();

// on another node, with more CPU:
commandBus.subscribe(CommandType.class.getName(), handler);
commandBus.subscribe(AnotherCommandType.class.getName(), handler2);
commandBus.updateLoadFactor(150); // defaults to 100
connector.connect();

// from now on, just deal with commandBus as if it is local...
```

> **Note**
>
> Note that it is not required that all segments have command handlers for the same type of commands. You may use different segments for different command types altogether. The distributed command bus will always choose a node to dispatch a command to that has support for that specific type of command.

## Configuration in Spring Boot

If you use Spring, you may want to consider using the `JGroupsConnectorFactoryBean`. It automatically connects the connector when the `ApplicationContext` is started, and does a proper disconnect when the `ApplicationContext` is shut down. Furthermore, it uses sensible defaults for a testing environment \(but should not be considered production ready\) and autowiring for the configuration.

The settings for the JGroups connector are all prefixed with `axon.distributed.jgroups`.

```text
# the address to bind this instance to. By default, it attempts to find the Global IP address
axon.distributed.jgroups.bind-addr=GLOBAL
# the port to bind the local instance to
axon.distributed.jgroups.bind-port=7800

# the name of the JGroups Cluster to connect to
axon.distributed.jgroups.cluster-name=Axon

# the JGroups Configuration file to configure JGroups with
axon.distributed.jgroups.configuration-file=default_tcp_gossip.xml

# The IP and port of the Gossip Servers (comma separated) to connect to
axon.distributed.jgroups.gossip.hosts=localhost[12001]
# when true, will start an embedded Gossip Server on bound to the port of the first mentioned gossip host.
axon.distributed.jgroups.gossip.auto-start=false
```

