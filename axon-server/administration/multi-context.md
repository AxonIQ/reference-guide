# Multi-Context

> Note: This feature is only available on the Enterprise Edition of AxonServer

To recap the definition as we have seen in the [clustering](multi-context.md) section, contexts allow for strong segregation of data without requiring deploying and managing full instances.

An Axon Server EE cluster can be setup to store events for multiple contexts. Each context has it own set of files \(containing Event/Snapshot data\) stored in a separate directory. The creation process involves the addition of member nodes within a cluster that will serve that context and the provisioning of a separate physical directory on each of those member nodes. The location of the newly created directory would be under the _root data directory_ of the member node.

A depiction of multiple registered contexts within an Axon Server EE cluster is shown below

![Multiple contexts within an Axon Server EE cluster](../../.gitbook/assets/multi-context.jpg)

The [clustering](multi-context.md) section details the creation of the \__admin and default contexts_ when a new Axon Server cluster is created. The _\_admin_ context **is used to process all configuration changes in Axon Server, so it contains the master configuration from which all contexts get their information. The** \__admin_ context does not have an event store and the configuration information is stored in a control database. The _default_ context is the context used by clients when they have not specified any context information. In case you would like to create a cluster without creating a default context, it is recommended to use the [Automatic-Initialization](multi-context.md) feature where you can control explicitly which contexts can be created or not.

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
$ java -jar ./axonserver-cli.jar register-context -S http://[node]:[port] -c [context-name] -n [members]‌
```

_Mandatory parameters_

* _**-c**_ refers to the context name. The context name must match the following regular expression "\[a-zA-Z\]\[a-zA-Z\_-0-9\]\*", so it should start with a letter \(uppercase or lowercase\), followed by a combination of letters, digits, hyphens and underscores.
* _**-n**_ refers to the comma separated list of node names that should be members of the new context. This parameter registers them as "PRIMARY" member nodes of that context

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context 
* _**-a**_ refers to the comma separated list of node names that should be "ACTIVE\_BACKUP" member nodes of that context
* _**-m**_ refers to the comma separated list of node names that should be "MESSAGING\_ONLY" member nodes of that context
* _**-p**_ refers to the comma separated list of node names that should be "PASSIVE\_BACKUP" member nodes of that context
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

_**Adding a node to a Context**_

The add-node-to-context command helps in the registration of a new member node creation of an existing context. A sample of the command with the mandatory parameters is depicted below

```text
$ java -jar ./axonserver-cli.jar add-node-to-context -S http://[node]:[port] -c [context-name] -r [role of the node] -n [node name]‌
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
$ java -jar ./axonserver-cli.jar delete-node-from-context -S http://[node]:[port] -c [context-name] -n [node name]‌
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

