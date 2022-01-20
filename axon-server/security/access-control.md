# Access Control

As Axon Server is an event store and may contain sensitive data it is always a good practice to enable access control in production and production-like environments. Enabling access control will require applications to provide a token when accessing Axon Server services \(both through gRPC and HTTP\), and require users to login to the dashboard.‌ In this section we will describe how to configure access control on both the Axon Server side as well as the Axon Framework side.

To enable access control in Axon Server \(SE/EE\) add the following property to `axonserver.properties`:

```text
axoniq.axonserver.accesscontrol.enabled=true
```

Because Axon Server SE deals with this differently than Axon Server EE, they will be addresses separately:

* [Axon Server SE](access-control-se.md#access-control-for-axon-server-se)
* [Axon Server EE](access-control-ee.md#access-control-for-axon-server-ee)
* [Axon Framework apps](access-control-clients.md#security-for-axon-framework-client-applications)
* [Axon Server CLI](access-control-cli.md#access-control-and-the-cli)
* [Direct access to the REST and gRPC APIs](access-control-api.md#access-control-on-the-rest-and-grpc-apis)

For Axon Server EE, we have additional sections on the external authentication extensions:

* [LDAP Extension](access-control-ldap.md#axon-server-ee-ldap-extension)
* [OAuth2 Extension](access-control-oauth2.md#axon-server-ee-oauth-extension)

## Using the CLI to create a user

If you haven't used the cluster template to create an initial user, you can use the CLI to create it. For this you will need an admin-level access token, as described [here](access-control-cli.md). To do this execute the "`register-user`" command:

```text
$ java -jar axonserver-cli.jar register-user
usage: register-user
 -i,--insecure-ssl         Do not check the certificate when connecting
                           using HTTPS.
    --no-password          [Optional] Create a (locked) user account
                           without a password.
 -o,--output <arg>         Output format (txt,json)
 -p,--password <arg>       [Optional] Password for the user
 -r,--roles <arg>          [Optional] roles for the user
 -S,--server <arg>         Server to send command to (default
                           http://localhost:8024)
 -s,--https                Use HTTPS to connect to the server, rather than
                           HTTP.
 -t,--access-token <arg>   [Optional] Access token to authenticate at
                           server
 -u,--username <arg>       Username
```

### Mandatory parameters

* `-u` or `--username` specifies the username.
* `-r` or `--roles` specifies the role of the user. Specify multiple roles by giving a comma separated list \(without spaces\), for example "`READ,ADMIN`". 

### Optional parameters

* `-p` or `--password` specifies the password of the user. If you do not specify a password with the "`-p`" option, the command line interface will prompt you for one. If you instead want a use account _without_ a password‌, for example when using Google OAuth2 authentication, use "`--no-password`".
* `--no-password` will cause the CLI to create a user acount with _no_ password set, which means you cannot login unless you use an external authentication provider.
* `-t` or `--access-token` specifies the access token to authenticate at the server to which the command is sent to. For SE this should be the same as [the (admin) token set in the properties](access-control-se.md). For EE this should be the security token discussed above.
* `-S` or `--server` can be used to specify the URL to the server that the command needs to be sent to. If this is not supplied it connects to "`http://localhost:8024`" by default.
* `-s` or `--https` will cause the CLI to use TLS, in effect changing the URL to "`https://localhost:8024`". Note that if you also want to change the port, you'll have to use "`-S`", in which case you can leave out "`-s`".
* `-i` or `--insecure-ssl` will tell the CLI that Axon Server is using a certificate which is not signed by a known CA, for example when using self-signed certificates.

Users can also be added using the REST API / UI Console that Axon Server SE provides. The CLI also allows the capabilities to [list all users](../administration/admin-configuration/command-line-interface.md#users) as well as [delete specific users](../administration/admin-configuration/command-line-interface.md#users).
