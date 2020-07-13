# Backup and Messaging-only Nodes

> **Note**
>
> This feature is only available in Axon Server Enterprise versions 4.3 and higher.

When you have larger clusters and specific requirements, you may want to have nodes acting in a different way within a context. For this reason, in Axon Server Enterprise 4.3, we have introduced a number of new roles that can be assigned to a node.

These are:

* _Primary nodes -&gt;_ This is the role for nodes prior to 4.3. Primary nodes handle client connections and messages, store events and may act as the leader for a context.
* _Backup nodes -&gt;_ These nodes maintain a copy of the event store, but will never become the leader of a context. There are two types of Backup nodes - [Active Backup](backup-and-messaging-only-nodes.md#active-backup-node) and [Passive Backup](backup-and-messaging-only-nodes.md#passive-backup-node)
* _Messaging-only nodes_ -&gt;  They handle client connections and all types of messages \(Commands/Queries/Events\) but they do not have an event store.

Each context needs to have at least one Primary node. This node is capable of becoming a leader and coordinating transactions across the context between the different member nodes. Even if you are not using Axon Server as an event store, you still need a leader as transactions also include making changes to the context configuration and access control lists.‌ It is also possible to [change the role](backup-and-messaging-only-nodes.md#changing-node-roles) that a particular node plays within a context.

## Backup nodes‌ <a id="backup-nodes"></a>

You can use backup nodes for instance when you want to ensure that you have a copy of your event store in another data center. As clients will never connect to a backup node and the backup node will never become the leader for a context it reduces the risk of high latency, compared to having a normal \(Primary\) node in another data center.‌ Just to reiterate, clients will never connect directly to a backup node.‌

Backup nodes come in two flavors

### _**Active Backup node**_

Active backup nodes maintain a real time copy of the Event Store by being _active_ _participants_ in transactions. To expand this, suppose a context within an Axon Server EE cluster has Active Backup nodes assigned to it \(in addition to the Primary nodes\). When an event is raised within a context the transaction to commit it in the Event Store is ready only if it receives a successful acknowledgement from at-least a certain number of those Active Backup nodes.

It is possible to customize the number of active backup nodes involved in the transactions by changing the value of the property `axoniq.axonserver.replication.min-active-backups` . The default value of this parameter is _**"1"**_ which means that if you have Active Backup nodes, at least one of them needs to be up at any time. The higher the value set for this property, the higher the number of Active backup member nodes that need to be available for a successful transaction, so this property affects the availability of the cluster and hence needs to be carefully managed.

There are three possible ways to assign the ACTIVE\_BACKUP role to a node within a context:

A\) The Axon Server EE UI Console. Navigate to the Contexts icon on the navigation menu of the console which will open up the context maintenance screen. The nodes can be added as an ACTIVE\_BACKUP role within a context.

B\) The _add-node-to-context_ command with the role option as ACTIVE\_BACKUP

```text
$ java -jar axonserver-cli.jar add-node-to-context  -S http://[node]:[port] -n [node name]‌ -c [context-name] -r PASSIVE_BACKUP
```

_Mandatory parameters_

* _**-c**_ refers to an existing context
* _**-n**_ refers to the node name that should be a member of this context
* _**-r as PASSIVE\_BACKUP**_ refers to the role of this node within the context 

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-t**_  refers to the access token to authenticate at server

C\) Axon Server EE provided REST API \(http:\[server\]:\[port\]/swagger-ui.html\) which offers the _context-rest-controller_ to help perform role maintenance operations

### _**Passive Backup node**_

Passive Backup nodes follow the primary nodes with on a best effort base. If they are disconnected for some time, it will not impact the overall availability of the context. Once the Passive backup nodes are connected again, they will update their event stores with the events that were added while they were gone. If you don't require the backup node to be fully up to date at any moment, you can configure one passive backup node.‌

There are three possible ways to assign the PASSIVE\_BACKUP role to a node within a context:

A\) The Axon Server EE UI Console. Navigate to the Contexts icon on the navigation menu of the console which will open up the context maintenance screen. The nodes can be added as a PASSIVE\_BACKUP role within a context.

B\) The _add-node-to-context_ command with the role option as PASSIVE\_BACKUP

```text
$ java -jar axonserver-cli.jar add-node-to-context  -S http://[node]:[port] -n [node name]‌ -c [context-name] -r PASSIVE_BACKUP
```

_Mandatory parameters_

* _**-c**_ refers to an existing context
* _**-n**_ refers to the node name that should be a member of this context
* _**-r as PASSIVE\_BACKUP**_ refers to the role of this node within the context 

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-t**_  refers to the access token to authenticate at server

C\) Axon Server EE provided REST API \(http:\[server\]:\[port\]/swagger-ui.html\) which offers the _context-rest-controller_ to help perform role maintenance operations

## Messaging-only nodes <a id="messaging-only-nodes"></a>

You can add nodes as messaging-only nodes to a context, if you don't want to use Axon Server as an event store, or if you want to have a large number of Axon Server nodes for a single context, without storing the events on each node. As the name already suggests, messaging-only nodes only route messages, they do not store events themselves. They do not participate in transactions and will clearly never become the leader for a context.‌

There are three possible ways to assign the MESSAGING\_ONLY role to a node within a context:

A\) The Axon Server EE UI Console. Navigate to the Contexts icon on the navigation menu of the console which will open up the context maintenance screen. The nodes can be added as a MESSAGING\_ONLY role within a context.

B\) The _add-node-to-context_ command with the role option as MESSAGING\_ONLY

```text
$ java -jar axonserver-cli.jar add-node-to-context  -S http://[node]:[port] -n [node name]‌ -c [context-name] -r MESSAGING_ONLY
```

_Mandatory parameters_

* _**-c**_ refers to an existing context
* _**-n**_ refers to the node name that should be a member of this context
* _**-r as MESSAGING\_ONLY**_ refers to the role of this node within the context 

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-t**_  refers to the access token to authenticate at server

C\) Axon Server EE provided REST API \(http:\[server\]:\[port\]/swagger-ui.html\) which offers the _context-rest-controller_ to help perform role maintenance operations

## Changing node roles <a id="changing-node-roles"></a>

Sometimes you may want to change the role a node has for a specific context. This may happen when you have a pre-existing cluster context configuration \(pre 4.3\) and now you want to be able to start using the new roles. The way to do this is to remove a node from a context and then add it again in the new role.‌

When you remove the node from the context you have an option to preserve the event store. Preserving the event store is recommended when you want to change the role for a node from _PRIMARY_ to _BACKUP_, or vice versa, as it would prevent a full replication of the event store when the node is added again with the new role.

There are three possible ways to change the role of a node within a context:

A\) The Axon Server EE UI Console. Navigate to the Contexts icon on the navigation menu of the console which will open up the context maintenance screen. You can choose to delete the specific node from the context \(using the delete icon\). In case you would like to preserve the event store, click on the check-box in the pop-up.

B\) The _delete-node-from-context_ command

```text
$ java -jar ./axonserver-cli.jar delete-node-from-context -S http://[node]:[port] -c [context-name] -n [node name]‌
```

_Mandatory parameters_

* _**-c**_ refers to an existing context
* _**-n**_ refers to the node name that should be a member of this context

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-t**_  refers to the access token to authenticate at server
* _**--preserve-event-store**_ removes the node from the context but leaves the event store files on that node.

C\) Axon Server EE provided REST API \(http:\[server\]:\[port\]/swagger-ui.html\) which offers the _context-rest-controller_ to help perform role maintenance operations

