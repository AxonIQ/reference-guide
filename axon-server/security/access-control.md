# Access Control

As Axon Server is an event store and may contain sensitive data it is always a good practice to enable access control in production and production-like environments. Enabling access control will require applications to provide a token when accessing Axon Server services \(both through gRPC and HTTP\), and require users to login to the dashboard.‌

To enable access control in Axon Server \(SE/EE\) add the following property to `axonserver.properties`:

```text
axoniq.axonserver.accesscontrol.enabled=true
```

## Axon Server Standard Edition <a id="axon-server-standard-edition"></a>

Axon Server Standard Edition provides a basic access control mechanism for:

* Client Applications connecting to Axon Server SE.
* Users accessing the Axon Server UI Console.

### Axon Framework Client Applications

Applications need to provide a valid token before they can perform all operations. For users there are two groups, \(1\) users that can login to the dashboard but cannot manage users and \(2\) users with `ADMIN` role, who are allowed to manage users.‌

The token that applications must provide to use Axon Server is defined in the `axonserver.properties` file.

```text
axoniq.axonserver.accesscontrol.token=[token]‌
```

The token that you set here must be used by all Axon Framework Applications connecting to Axon Server. The access token can be setup in the client using the property `axon.axonserver.token=[Token]`.

If you are using the REST APIs, you can specify the token in the HTTP requests via the following header:

```text
AxonIQ-Access-Token: my-token-value-here
```

### **Users - Axon Server SE UI Console**

You can create users either through the command line or through the dashboard web application. The _**initial user**_ has to be created through the command line as you would not be able to login to the dashboard otherwise.

To do this execute the _"register-user"_ command:

```text
$ java -jar axonserver-cli.jar register-user -u username -r roles [-p password] -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-u**_ refers to the username.
* _**-r**_ refers to the role of the user. Specify multiple roles by giving a comma separated list \(without spaces\) -&gt; READ,ADMIN. 
* _**-p**_ refers to the password of the user. If you do not specify a password with the -p option, the command line interface will prompt you for one.‌

_Optional parameters_

* _**-S**_ refers to the server to send the command to. If this is not supplied it connects to [http://localhost:8024](http://localhost:8024) by default. For Axon Server SE, the URL for the Axon Server SE will be the single running node.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to. This should be the same as the token setup in the `axonserver.properties` file.

Users can also be added using the REST API / UI Console that Axon Server SE provides. The CLI also allows the capabilities to [list all users](../administration/admin-configuration/command-line-interface.md#users) as well as [delete specific users](../administration/admin-configuration/command-line-interface.md#users).

## Axon Server Enterprise Edition <a id="axon-server-enterprise-edition"></a>

Axon Server Enterprise Edition provides a fine-grained access control mechanism for:

* Client Applications connecting to Axon Server SE.
* Users accessing the Axon Server UI Console.

In Axon Server EE we can grant specific roles to applications and users that will allow specific operations. Apart from just assigning the roles, you must also indicate for which context the role is granted, so that an application/user that has rights on only one context is not able to access data from other contexts.‌

A summary of the various roles is depicted below

| Role Name | Description |
| :--- | :--- |
| ADMIN | Administer the cluster, manage contexts, users and applications |
| CONTEXT\_ADMIN | Manage event processors within a specific context |
| DISPATCH\_COMMANDS | Dispatch commands |
| DISPATCH\_QUERY | Dispatch queries and subscription queries |
| MONITOR | View context information |
| PUBLISH\_EVENTS | Publish events and snapshots |
| READ\_EVENTS | Read events and snapshots from the event store |
| SUBSCRIBE\_COMMAND\_HANDLER | Register command handlers |
| SUBSCRIBE\_QUERY\_HANDLER | Register query handlers |
| USE\_CONTEXT | Perform all operations on a context |
| READ \(Deprecated\) | Read events and perform queries |
| WRITE \(Deprecated\) | Publish events and perform commands |

### Axon Framework Client Applications

Instead of setting a single token in `axonserver.properties`, you must now register applications with specific roles. Since every request requires a token or a valid user, there is a _**special way**_ of creating the first application and/or user. This first application can be created using the command line interface on an Axon Server node and the CLI has to be necessarily run from the installation directory of Axon Server.

To register an application and get an access token use the "register-application" command. This command returns the generated token to use. Note that this token is only generated once, if you lose it you must delete the application and register it again to get a new token. If you want to define the token yourself, you can provide one in the command line command using the `-T` flag:

```text
$ java -jar axonserver-cli.jar register-application -a name -r roles  [-d description] [-T apptoken] -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-a**_ \_\*\*\_refers to the name of the application.
* _**-r**_ refers to the role of the application. Specify multiple roles by giving a comma separated list \(without spaces\), e.g. READ,ADMIN. 

_Optional parameters_

* _**-d**_ refers to the description of the application.
* _**-S**_ refers to the server to send the command to and if not supplied connects by default to [http://localhost:8024](http://localhost:8024). The URL should be pointing to any node serving the _\_admin_ context within an Axon Server EE cluster.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to.
* _**-T**_ in case you want to define the token yourself for a newly registered application.

The ADMIN role is only valid for the \_admin context, the other roles are specific to another context. In addition to the role name you can also supply the context to which this role applies, for example _{role\_name}@{context\_name}_. Also if no context is mentioned in Axon Server EE, the role is granted to the application for all registered contexts, including contexts that are created after the role has been granted.

The token that you set here must be used by all Axon Framework Applications connecting to Axon Server. The access token can be setup in the client using the property `axon.axonserver.token=[Token]`

If you are using the REST APIs, you can specify the token in the HTTP requests via the following header:

```text
AxonIQ-Access-Token: my-token-value-here
```

### **Users - Axon Server EE UI Console**

When using Axon Server with access control enabled, users need to be defined to access the Axon Server EE Dashboard. The _**first user**_ has to be created using the command line interface on an Axon Server node and the CLI has to be necessarily run from the installation directory of Axon Server.

To do this execute the _"register-user"_ command:

```text
$ java -jar axonserver-cli.jar register-user -u username -r roles [-p password] -S http://[node]:[port] [-t token]
```

_Mandatory parameters_

* _**-u**_ refers to the username.
* _**-r**_ refers to the role of the user. Specify multiple roles by giving a comma separated list \(without spaces\).
* _**-p**_ refers to the password of the user. If you do not specify a password with the -p option, the command line interface will prompt you for one.‌

_Optional parameters_

* _**-S**_ refers to the server to send the command to. If this is not supplied it connects to [http://localhost:8024](http://localhost:8024) by default.
* _**-t**_  refers to the access token to authenticate at the server to which the command is sent to. This should be the same as the token setup in the `axonserver.properties` file.

Users can also be added using the REST API / UI Console that Axon Server EE provides. The CLI also allows the capabilities to [list all users](../administration/admin-configuration/command-line-interface.md#users) as well as [delete specific users](../administration/admin-configuration/command-line-interface.md#users).

### Axon Server cluster <a id="axon-server-cluster"></a>

If access control is enabled, the nodes in the cluster also need to attach a token with the messages sent between them. This token must be defined in the `axonserver.properties`file on each node with property `axoniq.axonserver.accesscontrol.internal-token`. The value of this property must be the same for all nodes and is comparable to the token for SE. If you have a running cluster and want to enable access control, be sure to first configure this token on all nodes, before you start enabling access control. That way you can keep the restarts limited to a single node at a time, and never lose full cluster availability.

