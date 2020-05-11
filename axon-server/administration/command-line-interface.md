# Command Line Interface

The Axon Server command line interface allows updating the Axon Server configuration through scripts or from a command line.‌

For the Axon Server Standard edition the only supported commands are:‌

* metrics
* users
* register-user
* delete-user
* purge-events \(only in [Development Mode](../installation/development-mode.md)\)

‌

Axon Server Enterprise edition supports these additional commands:‌

* applications
* register-application
* delete-application
* init-cluster
* cluster
* register-node
* unregister-node
* contexts
* register-context
* delete-context
* add-node-to-context
* delete-node-from-context

‌

The general format of command line commands is:

```text
java -jar axonserver-cli.jar <Command> -S <url-to-axonserver> <command options>
```

‌

Or when running from a bash-like shell:

```text
axonserver-cli.jar <Command> -S <url-to-axonserver> <command options>
```

‌

The option -S with the url to the Axon Server is optional, if it is omitted it defaults to [http://localhost:8024](http://localhost:8024/).‌

## Access control <a id="access-control"></a>

‌

When running Axon Server with access control enabled, executing commands remotely requires an access token. This has to provided with the -t option. When you run a command on the Axon Server node itself from the directory where Axon Server was started, you don't have to provide a token.‌

For Axon Server Standard Edition the token is specified in the axonserver.properties file \(property name = **axoniq.axonserver.token**\). In Enterprise Edition you need to register an application with ADMIN role, and you can use that application's token to perform command line commands remotely.‌

## Commands <a id="commands"></a>

‌

This section describes all commands supported by the command line interface, grouped by subject.‌

### Metrics <a id="metrics"></a>

‌

Usage:

```text
axonserver-cli.jar metrics
```

‌

Overview of all Axon specific metrics.‌

### User <a id="user"></a>

‌

When using Axon Server with access control enabled, users need to be defined to access the Axon Server Dashboard. Users with only READ role can view the information in Axon Server Dashboard but not make any changes, users with ADMIN role can make changes.‌

_users_‌

Usage:

```text
axonserver-cli.jar users
```

‌

Returns a list of all registered users and their roles.‌

_register-user_‌

Usage:

```text
axonserver-cli.jar register-user -u username -r roles [-p password]
```

‌

Registers a user with specified roles. Specify multiple roles by giving a comma separated list \(without spaces\), e.g. READ,ADMIN. If you do not specify a password with the -p option, the command line interface will prompt you for one.‌

_delete\_user_‌

Usage:

```text
axonserver-cli.jar delete-user -u username
```

‌

Deletes the specified user.‌

### Application \(Enterprise Edition only\) <a id="application-enterprise-edition-only"></a>

‌

_applications_‌

Usage:

```text
axonserver-cli.jar applications
```

‌

Lists all applications and the roles per application per context.‌

_register-application_‌

Usage:

```text
axonserver-cli.jar register-application -a name [-d description] [-r roles] [-T token]
```

‌

Registers an application with specified name. Roles is a comma seperated list of roles per context, where a role per context is the combination of &lt;Role&gt;@, e.g. READ@context1,WRITE@context2. If you do not specify the context for the role it will be for all contexts.‌

If you omit the -T option, Axon Server will generate a unique token for you. Applications must use this token to access Axon Server. Note that this token is only returned once, you will not be able to retrieve this token later.‌

_delete-application_‌

Usage:

```text
axonserver-cli.jar delete-application -a name
```

‌

Deletes the application from Axon Server. Applications will no longer be able to connect to Axon Server using this token.‌

### Cluster \(Enterprise Edition only\) <a id="cluster-enterprise-edition-only"></a>

‌

_cluster_‌

Usage:

```text
axonserver-cli.jar cluster
```

‌

Shows all the nodes in the cluster, including their hostnames, http ports and grpc ports.‌

_init-cluster_‌

Usage:

```text
axonserver-cli.jar init-cluster [-c context]
```

‌

Initializes the cluster, creates the _\_admin_ context and the specified context. If no context specified it creates _default_ context.‌

_register-node_‌

Usage:

```text
axonserver-cli.jar register-node -h node-in-cluster [-p internal-grpc-port] [-c context]
```

‌

Tells this Axon Server node to join a cluster. You need to specify the hostname and optionally the internal port number of a node that is the leader of the \_admin context. If you do not specify the internal port number it uses 8224 \(default internal port number for Axon Server\).‌

If you specify a context, the new node will be a member of the specified context. If you haven't specified a context, the new node will become a member of all defined contexts.‌

_unregister-node_‌

Usage:

```text
axonserver-cli.jar unregister-node -n nodename
```

‌

Removes the node with specified name from the cluster. After this, the node that is deleted will still be running in standalone mode.‌

### Context \(Enterprise Edition only\) <a id="context-enterprise-edition-only"></a>

‌

_contexts_‌

Usage:

```text
axonserver-cli.jar contexts
```

‌

Lists all contexts and the nodes assigned to the contexts. Per context it shows the master \(responsible for replicating events\) and the coordinator \(responsible for rebalancing\).‌

_register-context_‌

Usage:

```text
axonserver-cli.jar register_context -c name -n primary-nodes [-m messaging-only-nodes] [-a active-backup-nodes] [-p passive-backup-nodes]
```

‌

Creates a new context, with the specified nodes assigned to it. You have to specify at least one primary node.‌

_delete-context_‌

Usage:

```text
axonserver-cli.jar delete-context -c name
```

‌

Deletes a context.‌

_add-node-to-context_‌

Usage:

```text
axonserver-cli.jar add-node-to-context -c context -n nodename [-r PRIMARY|ACTIVE_BACKUP|PASSIVE_BACKUP|MESSAGING_ONLY]
```

‌

Adds the node with name to the specified context. If no role \(-r\) is specified the node will be added as a primary node.‌

_delete-node-from-context_‌

Usage:

```text
axonserver-cli.jar delete-node-from-context -c context -n nodename [--preserve-event-store]
```

‌

Deletes the node with name from the specified context. Deletes the event store on the specified node by default. If the preserve-event-store option is specified it will not delete the event store.

