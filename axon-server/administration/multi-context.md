# Multi-Context

> Note: This feature is only available on the Enterprise Edition of AxonServer

To recap the definition as we have seen in the [clustering](multi-context.md) section, contexts allow for strong segregation of data without requiring deploying and managing full instances.

An Axon Server EE cluster can be setup to store events for multiple contexts. Each context has it own set of files \(containing Event/Snapshot data\) stored in a separate directory. Axon Server will replicate the context data to a number of nodes depending on the replication group the context is assigned to. When you are creating a context you can either add it to an already existing replication group, or create a new replication group for this context. More on replication groups can be found in the [replication groups](replication-groups.md) section.

Each context has a separate physical directory on each of the member nodes. This directory is by default determined by the _**axoniq.axonserver.events.storage**_ and _**axoniq.axonserver.snapshot.storage**_ properties. Each context getting a subdirectory below the location specified by the property. Upon creation of a context you can specify a different location.

A depiction of multiple registered contexts within an Axon Server EE cluster is shown below

![Multiple contexts within an Axon Server EE cluster](../../.gitbook/assets/multi-context.jpg)

The [clustering](multi-context.md) section details the creation of the \__admin and default contexts_ when a new Axon Server cluster is created. The _\_admin_ context **is used to process all configuration changes in Axon Server, so it contains the master configuration from which all contexts get their information. The** \__admin_ context does not have an event store and the configuration information is stored in a control database. The _default_ context is the context used by clients when they have not specified any context information. In case you would like to create a cluster without creating a default context, it is recommended to use the [Automatic-Initialization](multi-context.md) feature where you can control explicitly which contexts can be created or not.

When you create a context, there are a number of default properties that you can override, specifically for the context. These are:

| property | default value | description |
| :--- | :--- | :--- |
| event.storage | \[axoniq.axonserver.event.storage\]/\[context-name\] | Location where the event store's event information will be stored |
| event.segment-size | 250MB | Size of the event store's event segments |
| event.index-format | JUMP\_SKIP\_INDEX | Index type used for the events in this context \(JUMP\_SKIP\_INDEX or BLOOM\_FILTER\_INDEX |
| event.max-bloom-filters-in-memory | 100 | Number of bloom filters for events to be kept in memory \(only applicable if index is BLOOM\_FILTER\_INDEX\) |
| event.max-indexes-in-memory | 50 | Number of index files for events to be kept in memory |
| event.retention-time | 7d | Retention time for the events in the primary locations \(if secondary location specified\) |
| snapshot.storage | \[axoniq.axonserver.snapshot.storage\]/\[context-name\] | Location where the event store's event information will be stored |
| snapshot.segment-size | 250MB | Size of the event store's snapshot segments |
| snapshot.index-format | JUMP\_SKIP\_INDEX | Index type used for the snapshots in this context \(JUMP\_SKIP\_INDEX or BLOOM\_FILTER\_INDEX |
| snapshot.max-bloom-filters-in-memory | 100 | Number of bloom filters for snapshots to be kept in memory \(only applicable if index is BLOOM\_FILTER\_INDEX\) |
| snapshot.max-indexes-in-memory | 50 | Number of index files for snapshots to be kept in memory |
| snapshot.retention-time | 7d | Retention time for the snapshots in the primary locations \(if secondary location specified\) |

### Index formats

As of version 4.4, Axon Server has a new format for the index of events and snapshots, called JUMP\_SKIP\_INDEX. This is the default format for all contexts that are created from this version onwards. This index format uses a global index to locate the last event for a specific aggregate, and maintains per segment per aggregate the location of the previous event. It improves the efficiency in looking up aggregates that are distributed over segments that are further apart. For instance if you have 2000 event segments and an aggregate has events in segment 1500, 1000 and 500, using this index, Axon Server will find the latest event using the global index, and then from the index for segment 1500 that the previous event is in segment 1000. This prevents checking \(the indexes of\) all the files in between. When using this index, Axon Server will no longer create bloom filter files. For existing contexts the index format will remain BLOOM\_FILTER\_INDEX.

> **Pre-4.4 Context Deletion**
>
> Note that when you delete an existing context with the preserve data option and then recreate it, without specifying the index format, Axon Server will use the JUMP\_SKIP\_INDEX format. This means that it will create a new index for the existing data, if the old format was BLOOM\_FILTER\_INDEX. Depending on the size of the event store this can take a long time.

## Multi-tier storage

Another feature that is available from Axon Server version 4.4 is multi-tier storage. With this option you can choose to keep only the most recent event store on your primary nodes, and keep the full event store on the secondary nodes. The primary nodes can have fast \(more expensive\) disks, and the secondary nodes can have less expensive disks. As most of the reads will typically be done on the most recent data, it could reduce storage cost without too much impact on performance. The time period for which evens will be kept on the primary nodes is set using the retention time properties. Axon Server provides global settings, which can be set in the axonserver.properties file, and it allows you to set the retention time on the context level. To use the multi-tier storage feature, the replication group for the context must have nodes with SECONDARY role. It is recommended to have at least 2 nodes with SECONDARY role, to prevent a single point of failure.

## Context Maintenance

The operational maintenance of contexts within an Axon Server EE cluster can be done via any one of the following provided utilities

* [CLI](multi-context.md#command-line-interface)  \(axonserver-cli.jar\) provided by Axon Server EE
* [UI Console](multi-context.md#user-interface) of Axon Server EE
* [REST API](multi-context.md#rest-api) provided by Axon Server EE

Let us deep dive into these capabilities in more detail.

### Command-Line Interface

Axon's command-line utility \(_axonserver-cli.jar - part of the Axon Server distributable_\) offers the following options to operate and maintain contexts

#### _**Creating context\(s\)**_

The register-context command helps in the registration and creation of a new context. A sample of the command with the mandatory parameters is depicted below

```text
$ java -jar ./axonserver-cli.jar register-context -S http://[node]:[port] -c [context-name] -n [members]
```

This will create a new replication group with the name of the context, with the specified member nodes, and creates a context in this replication group.

Another example:

```text
$ java -jar ./axonserver-cli.jar register-context -S http://[node]:[port] -c [context-name] -g [replication-group] -prop event.storage=[location]  -prop snapshot.storage=[location]
```

This creates a new context in an already existing replication group. Event and snapshot files are stored in the specified location.

_Mandatory parameters_

* _**-c**_ refers to the context name. The context name must match the following regular expression "\[a-zA-Z\]\[a-zA-Z\_-0-9\]\*", so it should start with a letter \(uppercase or lowercase\), followed by a combination of letters, digits, hyphens and underscores.

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-g**_ refers to the name of the replication group where the context will be added to
* _**-n**_ refers to the comma separated list of node names that should be members of the new context. This parameter registers them as "PRIMARY" member nodes of that context
* _**-a**_ refers to the comma separated list of node names that should be "ACTIVE\_BACKUP" member nodes of that context
* _**-m**_ refers to the comma separated list of node names that should be "MESSAGING\_ONLY" member nodes of that context
* _**-p**_ refers to the comma separated list of node names that should be "PASSIVE\_BACKUP" member nodes of that context
* _**-s**_ refers to the comma separated list of node names that should be "SECONDARY" member nodes of that context
* _**-prop**_ refers to properties that can be set for the new context. The value should be in the form \=\
* _**-t**_  refers to the access token to authenticate at server

_**Deleting context\(s\)**_

The delete-context command helps in the deletion of a context and its associated data from all member nodes of that context. A sample of the command with the mandatory parameters is depicted below

```text
$ java -jar ./axonserver-cli.jar delete-context -S http://[node]:[port] -c [context-name]
```

_Mandatory parameters_

* _**-c**_ refers to the context that needs to be deleted

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-t**_  refers to the access token to authenticate at server
* _**--preserve-event-store"**_  option to keep the event store files when deleting the context \(Axon Server deletes the event files by default\)

_**Adding a node to a Context**_

The add-node-to-context command helps in the registration of a new member node creation of an existing context. A sample of the command with the mandatory parameters is depicted below

```text
$ java -jar ./axonserver-cli.jar add-node-to-context -S http://[node]:[port] -c [context-name] -r [role of the node] -n [node name]
```

_Mandatory parameters_

* _**-c**_ refers to an existing context
* _**-n**_ refers to the node name that should be a member of this context
* _**-r**_ refers to the role of this node within the context 

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-t**_  refers to the access token to authenticate at server

_**Deleting a node from a context**_

The delete-node-from-context command helps in the deletion member node from an existing context. A sample of the command with the mandatory parameters is depicted below

```text
$ java -jar ./axonserver-cli.jar delete-node-from-context -S http://[node]:[port] -c [context-name] -n [node name]
```

_Mandatory parameters_

* _**-c**_ refers to an existing context
* _**-n**_ refers to the node name that should no longer be a member of this context

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-t**_  refers to the access token to authenticate at server
* _**--preserve-event-store**_ removes the node from the context but leaves the event store files on that node.

_**List all contexts**_

The contexts command lists down all the contexts registered within the cluster, including its name, the leader member node of the context and all the member nodes within the context

```text
$ java -jar ./axonserver-cli.jar contexts
```

_Optional parameters_

* _**-o json**_ will display the output in a json format

### User Interface

Another option to maintain contexts is via the UI console of Axon Server EE. Navigate to the Contexts icon on the navigation menu of the console which will open up the context maintenance screen. The operations listed above are possible through the console.

### REST API

Axon Server EE provides a REST API to perform context maintenance operations. The API is accessible at http:\[server\]:\[port\]/swagger-ui.html and offers the _context-rest-controller_ to help perform context maintenance operations

