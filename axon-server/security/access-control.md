# Access Control

As Axon Server is an event store and may contain sensitive data it is always a good practice to enable access control in production and production-like environments. Enabling access control will require applications to provide a token when accessing Axon Server services \(both through gRPC and HTTP\), and require users to login to the dashboard.‌

To enable access control in Axon Server add the following property to `axonserver.properties`:

```text
axoniq.axonserver.accesscontrol.enabled=true
```

## Axon Server Standard Edition <a id="axon-server-standard-edition"></a>

‌

Axon Server Standard Edition has a basic access control mechanism, where applications that provide a valid token can perform all operations. For users there are two groups, users that can login to the dashboard but cannot manage users and users with ADMIN role, who are allowed to manage users.‌

The token that applications must provide to use Axon Server is defined in the `axonserver.properties` file.

```text
axoniq.axonserver.accesscontrol.token=[token]
```

‌

The token that you set here must be used by all applications connecting to Axon Server.‌

You can create users either through the command line or through the dashboard web application. The initial user has to be created through the command line as you would not be able to login tp the dashboard. To do this execute the following command:

```text
$ java -jar axonserver-cli.jar register-user -S        http://[node]:[port] -u [username] -r [roles] -t [token]
```

‌

The token that you must use here is the token you have set in the `axonserver.properties` file. In this case you will be prompted for a password \(you can pass the password in the command line using the -p option\). Valid roles are READ and ADMIN.‌

## Axon Server Enterprise Edition <a id="axon-server-enterprise-edition"></a>

‌

Axon Server Enterprise Edition has a much more fine-grained access control mechanism. Here you can grant specific roles to application that will allow specific operations. In this edition you can also grant more specific roles to users. Apart from just assigning the roles, you must also indicate for which context the role is granted, so that an application that has rights on only one context is not able to access data from other contexts.‌

Instead of setting a single token in `axonserver.properties`, you must now register applications with specific roles. Since every request requires a token or a valid user, there is a special way of creating the first application and/or user. You can create those using the command line interface on an Axon Server node, from the same directory where Axon Server was started.‌

To register an application and get an access token use the following command:

```text
$ java -jar axonserver-cli.jar register-application -S        http://[node]:[port] -a [name] -d [description] -r [roles]
```

‌

Roles may be a comma separated list of roles or if you want to grant multiple roles you can repeat the -r for each role.‌

This command returns the generated token to use. Note that this token is only generated once, if you loose it you must delete the application and register it again to get a new token. If you want to define the token yourself, you can provide one in the command line command using the `-T` flag, e.g.:

```text
$ java -jar axonserver-cli.jar register-application -a        [name] -d [description] -r [roles] -T [token-for-app]
```

‌

The minimum length for a token is 8 characters.‌

### Roles <a id="roles"></a>

‌

You can grant the following roles to users and applications:‌

* ADMIN - administer the cluster, manage contexts, users and applications.
* CONTEXT\_ADMIN - manage event processors within a specific context
* DISPATCH\_COMMANDS - dispatch commands
* DISPATCH\_QUERY - dispatch queries and subscription queries
* MONITOR - view context information
* PUBLISH\_EVENTS - publish events and snapshots
* READ\_EVENTS - read events and snapshots from eventstore
* SUBSCRIBE\_COMMAND\_HANDLER - register command handlers
* SUBSCRIBE\_QUERY\_HANDLER - register query handlers
* USE\_CONTEXT - perform all operations on a context
* READ \(deprecated\) - read events and perform queries
* WRITE \(deprecated\) - publish events and perform commands

‌

The ADMIN role is only valid for the \_admin context, the other roles are specific to another context. To grant a role in a specific context use role@context. If you don't specify the @context part, Axon Server grants the role to all contexts, including contexts that you create after the role has been granted.‌

### Axon Server cluster <a id="axon-server-cluster"></a>

‌

If access control is enabled, the nodes in the cluster also need to attach a token with the messages sent between them. This token must be defined in the `axonserver.properties`file on each node with property `axoniq.axonserver.accesscontrol.internal-token`. The value of this property must be the same for all nodes.‌

## Axon Framework applications <a id="axon-framework-applications"></a>

‌

Specify the access token in the client by setting the property:‌

* `axon.axonserver.token=[Token]`

‌

If you are using the REST APIs, you can specify the token in the HTTP requests via the following header:

```text
AxonIQ-Access-Token: my-token-value-here
```

