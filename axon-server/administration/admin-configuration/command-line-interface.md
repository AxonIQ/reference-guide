# Command Line Interface

The Axon Server provides a _command line utility_ which allows for updating the Axon Server configuration through scripts or from the command line interface.‌ The utility is available as a jar file \(axonserver-cli.jar\) which is available as part of the Axon Server distributable \(SE/EE\).

## Quick Summary

A quick summary of the various commands is depicted below. Each command has a specific [format](command-line-interface.md#format) to it and [access control](command-line-interface.md#access-control) can also be enabled on it for security purposes.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Area (Server Edition)</th>
      <th style="text-align:left">Command-Line Options</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p><a href="command-line-interface.md#user"><em><b>Users</b></em></a>
        </p>
        <p><em>(Standard/Enterprise)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">users</td>
      <td style="text-align:left">Lists all users</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">register-user</td>
      <td style="text-align:left">Registers a user within an Axon Server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">delete-user</td>
      <td style="text-align:left">Deletes a user within Axon Server</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="command-line-interface.md#metrics"><em><b>Metrics</b></em></a>
        </p>
        <p><em>(Standard/Enterprise)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">metrics</td>
      <td style="text-align:left">Lists all metrics for an Axon Server</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="command-line-interface.md#application-enterprise-edition-only"><em><b>Applications</b></em></a>
        </p>
        <p><em>(Enterprise Only)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">applications</td>
      <td style="text-align:left">Lists all applications within an Axon Server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">register-application</td>
      <td style="text-align:left">Registers an application with Axon Server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">delete-application</td>
      <td style="text-align:left">Deletes a registered application</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="command-line-interface.md#cluster-enterprise-edition-only"><em><b>Cluster</b></em></a>
        </p>
        <p><em>(Enterprise Only)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster</td>
      <td style="text-align:left">Lists all the details of a cluster within an Axon Server EE deployment</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">init-cluster</td>
      <td style="text-align:left">Initiates a cluster within an Axon Server EE deployment</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">register-node</td>
      <td style="text-align:left">Registers a node as a member within a cluster</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">unregister-node</td>
      <td style="text-align:left">Unregisters a member node within a cluster</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">update-license</td>
      <td style="text-align:left">Uploads a new license file to the cluster</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="command-line-interface.md#replication-groups-enterprise-edition-only"><em><b>Replication Group</b></em></a>
        </p>
        <p><em>(Enterprise Only)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication-groups</td>
      <td style="text-align:left">Lists all details of registered replication groups within an Axon Server EE deployment</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">add-node-to-replication-group</td>
      <td style="text-align:left">Adds a node as a member of a replication group</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">register-replication-group</td>
      <td style="text-align:left">Creates a new replication group</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">delete-node-from-replication-group</td>
      <td style="text-align:left">Unregisters a member node of a replication group</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">delete-replication-group</td>
      <td style="text-align:left">Deletes a replication group</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="command-line-interface.md#context-enterprise-edition-only"><em><b>Context</b></em></a>
        </p>
        <p><em>(Enterprise Only)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">contexts</td>
      <td style="text-align:left">Lists all details of registered contexts within an Axon Server EE deployment</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">register-context</td>
      <td style="text-align:left">Creates a new context</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">delete-context</td>
      <td style="text-align:left">Deletes a context</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><em><b>Other</b></em>
        </p>
        <p><em>(Standard Only)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">purge-events</td>
      <td style="text-align:left">Purges events from an Event Store</td>
    </tr>
  </tbody>
</table>

### ‌Format

The general format of any command line command is:

```text
java -jar axonserver-cli.jar <Command> <command options> -S <server-to-send-command-to> -t <auth_token>
```

Or when running from a bash-like shell:

```text
axonserver-cli.jar <Command>  <command options> -S <server-to-send-command-to> -t <auth_token>
```

The option -S with the url to the Axon Server is optional, if it is omitted it defaults to [http://localhost:8024](http://localhost:8024/).‌ While for Axon Server SE, the URL for the Axon Server SE will be the single running node, for Axon Server EE, the URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.

### Access control

When running Axon Server with access control enabled, executing commands remotely requires an access token. This needs to be provided with the -t option. When you run a command on the Axon Server node itself from the directory where Axon Server was started, you don't have to provide a token.‌

For Axon Server Standard Edition the token is specified in the axonserver.properties file \(property name = _axoniq.axonserver.token_\). In Enterprise Edition you need to register an application with ADMIN role, and you can use that application's token to perform command line commands remotely.‌ The token needs to be supplied using the _**-t**_ option in any of the commands.

## Commands

This section describes all commands supported by the command line interface, grouped by the specific area.‌

### Users <a id="user"></a>

When using Axon Server with access control enabled, users need to be defined to access the Axon Server Console's Dashboard. Users with only READ role can view the information in the console dashboard but not make any changes, users with ADMIN role can make changes.‌

#### _**users**_**‌**

Returns a list of all registered users and their roles.‌

```text
$ java -jar axonserver-cli.jar users -S http://[node]:[port] -t [token]
```

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). For Axon Server SE, the URL for the Axon Server SE will be the single running node, while for Axon Server EE the URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**register-user**_**‌**

Registers a user with specified roles. For Axon Server SE, the only two roles possible are READ/ADMIN while for Axon Server EE, the following roles can be granted - ADMIN/CONTEXT\_ADMIN/DISPATCH\_COMMANDS/DISPATCH\_QUERY/MONITOR/PUBLISH\_EVENTS/READ\_EVENTS/SUBSCRIBE\_COMMAND\_HANDLER/SUBSCRIBE\_QUERY\_HANDLER/USE\_CONTEXT

In addition to the role name you can also supply the context to which this role applies like _{role\_name}@{context\_name}_. For Axon Server SE, the only context available is the _default_ context so the role will only apply to that context \(hence not necessary to supply the context name\). For Axon Server EE, the specific context can be included as required. Also if no context is mentioned in Axon Server EE, the role is granted to the user for all registered contexts.

```text
$ java -jar axonserver-cli.jar register-user -u username -r roles [-p password] -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-u**_ refers to the username.
* _**-r**_ refers to the role of the user. Specify multiple roles by giving a comma separated list \(without spaces\), e.g. READ,ADMIN. 
* _**-p**_ refers to the password of the user. If you do not specify a password with the -p option, the command line interface will prompt you for one.‌

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). For Axon Server SE, the URL for the Axon Server SE will be the single running node, while for Axon Server EE the URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**delete-user‌**_

Deletes the specified user.

```text
$ java -jar axonserver-cli.jar delete-user -u username -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-u**_ refers to the username of the user that needs to be deleted.

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). For Axon Server SE, the URL for the Axon Server SE will be the single running node, while for Axon Server EE the URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

### Metrics <a id="metrics"></a>

Overview of all Axon specific metrics.‌

```text
$ java -jar axonserver-cli.jar metrics -S http://[node]:[port] [-t token]
```

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). For Axon Server SE, the URL for the Axon Server SE will be the single running node, while for Axon Server EE the URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

### Applications \(Enterprise Edition only\) <a id="application-enterprise-edition-only"></a>

_**applications**_**‌**

Lists all applications and the roles per application per context.‌

```text
$ java -jar axonserver-cli.jar applications -S http://[node]:[port] [-t token]
```

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**register-application**_**‌**

Registers an application with specified name and role. The following roles can be granted - ADMIN/CONTEXT\_ADMIN/DISPATCH\_COMMANDS/DISPATCH\_QUERY/MONITOR/PUBLISH\_EVENTS/READ\_EVENTS/SUBSCRIBE\_COMMAND\_HANDLER/SUBSCRIBE\_QUERY\_HANDLER/USE\_CONTEXT

In addition to the role name you can also supply the context to which this role applies like _{role\_name}@{context\_name}_. Also if no context is mentioned in Axon Server EE, the role is granted to the application for all registered contexts.

This command returns the generated token to use. Note that this token is only generated once, if you lose it you must delete the application and register it again to get a new token. If you want to define the token yourself, you can provide one in the command line command using the `-T` flag, e.g.:

```text
$ java -jar axonserver-cli.jar register-application -a name -r roles  [-d description] [-T apptoken] -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-a**_ \_\*\*\_refers to the name of the application
* _**-r**_ refers to the role of the application. Specify multiple roles by giving a comma separated list \(without spaces\), e.g. READ,ADMIN. 

_Optional parameters_

* _**-d**_ refers to the description of the application.
* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.
* _**-T**_ in case you want to define the token yourself for newly registered application.

_**delete-application**_**‌**

Deletes the application from Axon Server EE

```text
$ java -jar axonserver-cli.jar delete-application -a name -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-a**_ \_\*\*\_refers to the name of the application

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

### Cluster \(Enterprise Edition only\)‌ <a id="cluster-enterprise-edition-only"></a>

_**cluster**_**‌**

Shows all the nodes in the cluster, including their hostnames, http ports and grpc ports.‌

```text
$ java -jar axonserver-cli.jar cluster -S http://[node]:[port] [-t token]
```

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**init-cluster**_**‌**

Initializes the cluster, creates the _\_admin_ context and the specified context.

```text
$ java -jar axonserver-cli.jar init-cluster [-c context] -S http://[node]:[port] [-t token]
```

_Optional parameters_

* _**-c**_ refers to the context that needs to be created along with the _\_admin_ context. If no context is specified the _default_ context is created
* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**register-node**_**‌**

Registers an Axon Server node with a cluster.

```text
$ java -jar axonserver-cli.jar register-node -h node-internal-host-name [-p internal-grpc-port] [-c context] -S http://[node]:[port] [-t token]
```

If you specify a context, the new node will be a member of the specified context. If you haven't specified a context, the new node will become a member of all defined contexts.‌

_Mandatory parameters_

* _**-h**_ refers to the internal host name of the node that needs to be added to the cluster.

_Optional parameters_

* _**-p**_ refers to the internal gRPC port of the node that needs to be added to the cluster. By default it is 8224.
* _**-c**_ refers to the context which this axon server node will be a member of. If no context is specified, the new node will become a member of all defined contexts.
* _**--no-contexts**_ will add the node to the cluster but will not register it to any of the defined contexts.
* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**unregister-node**_**‌**

Removes the node with specified name from the cluster. After this, the node that is deleted will still be running in standalone mode.‌

```text
$ java -jar axonserver-cli.jar unregister-node -n nodename‌ -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-n**_ refers to the name of the node that needs to be removed from the cluster.

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**update-license**_**‌**

Uploads a new license file to the cluster. Axon Server distributes the new license file to all nodes in the cluster.

```text
$ java -jar axonserver-cli.jar update-license -f [license-file] -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-n**_ refers to the name of the node that needs to be removed from the cluster.
* _**-f**_ refers to the file containing the license to update.

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

### Replication Groups \(Enterprise Edition only\) <a id="replication-groups-enterprise-edition-only"></a>

_**replication-groups**_**‌**

Lists all replication groups and the nodes assigned to the replication groups. For each replication groups it shows the name of the replication group, 
the master node for the replication group and the member nodes of the replication group.‌

```text
$ java -jar axonserver-cli.jar replication-groups -S http://[node]:[port] [-t token]
```

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.
* _**-o json**_ will display the output in a json format.

_**register-replication-group**_**‌**

The register-replication-group command helps in the registration and creation of a new replication group. 
A sample of the command with the mandatory parameters is depicted below:

```text
$ java -jar ./axonserver-cli.jar register-replication-group  -c [context-name] -n [members]‌ -a [members] -m [members] -p [members] -s [members] -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-c**_ refers to the replication group name. The replication group name must match the following regular expression "\[a-zA-Z\]\[a-zA-Z\_-0-9\]\*", so it should start with a letter \(uppercase or lowercase\), followed by a combination of letters, digits, hyphens and underscores.
* _**-n**_ refers to the comma separated list of node names that should be members of the new replication group. This parameter registers them as "PRIMARY" member nodes of that context.

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context.
* _**-a**_ refers to the comma separated list of node names that should be "ACTIVE\_BACKUP" member nodes of that replication group.
* _**-m**_ refers to the comma separated list of node names that should be "MESSAGING\_ONLY" member nodes of that replication group.
* _**-p**_ refers to the comma separated list of node names that should be "PASSIVE\_BACKUP" member nodes of that replication group.
* _**-p**_ refers to the comma separated list of node names that should be "SECONDARY" member nodes of that replication group.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**delete-replication-group**_**‌**

The delete-replication-group command helps in the deletion of a replication group and its associated data from all member nodes of that replication group. 
A sample of the command with the mandatory parameters is depicted below:

```text
$ java -jar ./axonserver-cli.jar delete-replication-group  -g [replication-group-name] -S http://[node]:[port]
```

_Mandatory parameters_

* _**-g**_ refers to the replication group that needs to be deleted.

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context.
* _**-t**_  refers to the access token to authenticate at  the server to which the command is sent to.
* _** --preserve-event-store**_  option to keep all the event store data for all the nodes in the replication group (false by default)

_**add-node-to-replication-group**_**‌**

The add-node-to-replication-group command helps in the registration of a new member node creation of an existing replication group.

```text
$ java -jar ./axonserver-cli.jar add-node-to-replication-group -g [replication-group-name] -r [role of the node] -n [node name] -S http://[node]:[port] [-t token]‌
```

_Mandatory parameters_

* _**-g**_ refers to an existing replication group.
* _**-n**_ refers to the node name that should be a member of this replication group.
* _**-r**_ refers to the role of this node within the replication group \(PRIMARY/MESSAGING\_ONLY/ACTIVE\_BACKUP/PASSIVE\_BACKUP/SECONDARY\).

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context.
* _**-t**_  refers to the access token to authenticate at  the server to which the command is sent to.

_**delete-node-from-replication-group**_**‌**

The delete-node-from-replication-group command helps in the deletion member node from an existing replication group. A sample of the command with the mandatory parameters is depicted below:

```text
$ java -jar ./axonserver-cli.jar delete-node-from-replication-group  -g [replication-group-name] -n [node name] -S http://[node]:[port] [-t token]‌
```

_Mandatory parameters_

* _**-g**_ refers to an existing replication group.
* _**-n**_ refers to the node name that should no longer be a member of this context.

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context.
* _**-t**_  refers to the access token to authenticate at  the server to which the command is sent to.
* _**--preserve-event-store**_ removes the node from the replication group but leaves the event store files on that node.

### Contexts \(Enterprise Edition only\) <a id="context-enterprise-edition-only"></a>

_**contexts**_**‌**

Lists all contexts and the nodes assigned to the contexts. For each context it shows the name of the context, the master node for the context and the member nodes of the context.‌

```text
$ java -jar axonserver-cli.jar contexts -S http://[node]:[port] [-t token]
```

_Optional parameters_

* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.
* _**-o json**_ will display the output in a json format.

_**register-context**_**‌**

The register-context command helps in the registration and creation of a new context. A sample of the command with the mandatory parameters is depicted below:

```text
$ java -jar ./axonserver-cli.jar register-context  -c [context-name] -g [replication-group-name] -n [members]‌ -a [members] -m [members] -p [members] -S http://[node]:[port] [-t token]
```

If you don't provide an existing replication group name, you need to provide the names and roles of the nodes to include in the replication group to create.
If you don't provide a replication group name, but do provide nodes, it will create a replication group with the same name as the context. 

_Mandatory parameters_

* _**-c**_ refers to the context name. The context name must match the following regular expression "\[a-zA-Z\]\[a-zA-Z\_-0-9\]\*", so it should start with a letter \(uppercase or lowercase\), followed by a combination of letters, digits, hyphens and underscores.

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context.
* _**-g**_ refers to the name of the replication group
* _**-n**_ refers to the comma separated list of node names that should be members of the new context. This parameter registers them as "PRIMARY" member nodes of that context.
* _**-a**_ refers to the comma separated list of node names that should be "ACTIVE\_BACKUP" member nodes of that context.
* _**-m**_ refers to the comma separated list of node names that should be "MESSAGING\_ONLY" member nodes of that context.
* _**-p**_ refers to the comma separated list of node names that should be "PASSIVE\_BACKUP" member nodes of that context.
* _**-s**_ refers to the comma separated list of node names that should be "SECONDARY" member nodes of that context.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.

_**delete-context**_**‌**

The delete-context command helps in the deletion of a context and its associated data from all member nodes of that context. A sample of the command with the mandatory parameters is depicted below:

```text
$ java -jar ./axonserver-cli.jar delete-context  -c [context-name] -S http://[node]:[port]
```

_Mandatory parameters_

* _**-c**_ refers to the context that needs to be deleted.

_Optional parameters_

* _**-S**_ if not supplied connects by default to [http://localhost:8024](http://localhost:8024). If supplied, it should be any node serving the _\_admin_ context.
* _**-t**_  refers to the access token to authenticate at  the server to which the command is sent to.
* _** --preserve-event-store**_  option to keep the event store data (false by default).

