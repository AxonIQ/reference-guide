# Configuring nodes with other roles

> **Note**
>
> This feature is only available in Axon Server Enterprise versions 4.3 and higher.

When you have larger clusters and specific requirements, you may want to have nodes acting in a different way within a context.
For this reason, we have introduced a number of new roles for nodes in Axon Server Enterprise 4.3:

- Primary node, this is the role for nodes prior to 4.3. Primary nodes handle client connections and messages, store events 
and may act as leader for a context.
- Backup node, maintain a copy of the event store, but will never become leader.
- Messaging-only node, handles client connections and messages but does not have an event store.

Each context needs at least one primary node. This node is capable of becoming a leader and coordinating transactions across the context. 
Even if you are not using Axon Server as an event store, you still need a leader as transactions also include making changes to the context 
configuration and access control lists.   

## Backup nodes

You can use backup nodes for instance when you want to ensure that you have a copy of your event store in another data center. As clients will 
never connect to a backup node and the backup node will never become leader for a context it reduces the risk of high latency, compared to having
a normal (primary) node in another data center.

Clients will never connect directly to a backup node.  

Backup nodes come in two flavors, active and passive backup nodes. Active backup nodes participate in transactions, so when you
store an event it is guaranteed to be in at least one backup node, before it is committed. This means that if you have active backup nodes, at least one 
of them needs to be up at any time. Since you may want to be able to perform maintenance on backup nodes without impacting availability of your Axon Server 
cluster you should always have more than one active backup node, if you decide to use active backup nodes.

Passive backup nodes follow the primary nodes with best effort. If they are disconnected for some time, it will not impact the overall availability of the
context. Once the passive backup nodes are connected again, they will update their event stores with the events that were added while they were gone.
If you don't require the backup node to be fully up to date at any moment, you can configure one passive backup node.

To add a node as a backup node to a context you can use the Axon Dashboard, or you can use the following command line command:

```text
java -jar axonserver-cli.jar add-node-to-context  -S http://axonserver:8024 -n my-backup-de -c my-context -r ACTIVE_BACKUP
``` 

or
```text
java -jar axonserver-cli.jar add-node-to-context  -S http://axonserver:8024 -n my-backup-de -c my-context -r PASSIVE_BACKUP
``` 

_To decide: do we want to suppport changing node roles between PRIMARY to BACKUP and between ACTIVE_BACKUP and PASSIVE_BACKUP?_

## Messaging-only nodes

You can add nodes as messaging-only nodes to a context, if you don't want to use Axon Server as an event store, or if you want to have a large number of 
Axon Server nodes for a single context, without storing the events on each node. 
As the name already suggests, messaging-only nodes only route messages, they do not store events themselves. They do not participate in transactions and 
will clearly never become leader for a context.   

To add a node as a messaging-only node to a context you can use the Axon Dashboard, or you can use the following command line command:

```text
java -jar axonserver-cli.jar add-node-to-context  -S http://axonserver:8024 -n my-backup-de -c my-context -r MESSAGING_ONLY
``` 
