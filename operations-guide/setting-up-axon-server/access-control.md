# Access Control

To enable access control add the following property to the `axonserver.properties`:

* `axoniq.axonserver.accesscontrol.enabled=true`

To register an application and get an access token use the following command:

```text
$ java -jar axonserver-cli.jar register-application -S 
       http://[node]:[port] -a applicationname -d description -r READ,WRITE,ADMIN
```

The address of the server specified in this command is the address of the current master node. The master will distribute the applications to all the replicas.

This command returns the generated token to use. Note that this token is only generated once, if you loose it you must delete the application and register it again to get a new token. If you want to define the token yourself, you can provide one in the command line command using the `-T` flag, e.g.:

```text
$ java -jar axonserver-cli.jar register-application -a 
       applicationname -d description -r READ,WRITE -T this-is-my-token
```

The minimum length for a token is 8 characters.

Specify the access token in the client by setting the property:

* `axon.axonserver.token=[Token]`

In the Free Edition it is not possible to create applications. If you want to use access control in this edition specify the property `axoniq.axonserver.accesscontrol.token` with any value you want on the Axon server and set the same value in the `axoniq.axonserver.token` property on the client.

You can access the Axon webpages when access control is enabled by providing a username and password. Users are created through the command line using the following command:

```text
$ java -jar axonserver-cli.jar register-user -S http://[node]:[port]
     -u USERNAME -r USER,ADMIN
```

The command will prompt for a password. If you want to set the password directly you can use the `-p` option.

Note that the command line interface also requires a token when executing remote commands. If you execute a command from an Axon Server node itself, you do not need to provide a token.

If access control is enabled and Axon Server is running in clustered mode, the nodes in the cluster also need to attach a token with the messages sent between them. This token must be defined in the application properties file on each node with property `axoniq.axonserver.accesscontrol.internal-token`. The value of this property must be the same for all nodes.
