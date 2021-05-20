# Access Control for Axon Server EE

When you set the "`axoniq.axonserver.accesscontrol.enabled`" property to "`true`", Axon Server will require a token or user account for access to its APIs.

## Tokens in Axon Server EE

In Axon Server EE, there are three types of tokens that can be defined:

| Type | Property | Purpose |
| :--- | :--- | :--- |
| Internal | `axoniq.axonserver.accesscontrol.internal-token` | Define a token for nodes in the cluster to authenticate each other. |
| System | `axoniq.axonserver.accesscontrol.systemtokenfile` | Define a file for a token with administrative rights. Default value is "`./security/.token`". |
| Application | _(Through the UI or CLI)_ | Applications are registered using the UI or CLI and assigned roles. See below for more details. |

Axon Server will generate the system token for the CLI if none is found, and the CLI (if started in the Axon Server working directory) will know the default location.

Axon Framework based applications should only need the non-admin token. If you whish to use tools to access the REST API directly, you must add an HTTP header named "`AxonIQ-Access-Token`", as in the following example:

```bash
$ curl -H 'AxonIQ-Access-Token: system-token' -s http://localhost:8024/v1/public/context | jq '.[] | select(.context=="default")'
{
  "context": "default",
  "replicationGroup": "default",
  "metaData": {
    "event.index-format": "JUMP_SKIP_INDEX",
    "snapshot.index-format": "JUMP_SKIP_INDEX"
  },
  "changePending": false,
  "pendingSince": 0,
  "leader": "e32c48ab5047",
  "roles": [
    {
      "node": "e32c48ab5047",
      "role": "PRIMARY"
    }
  ]
}
```

## User Accounts in Axon Server EE

When you create a user account, you assign roles, which will determine the user's access rights. The user accounts are generally only used for the UI, although they are also valid for access to the REST API, using Basic Authentication. In contrast to SE, where you could only choose between "normal" and "admin" accounts, you can now assign several roles from a long list, and each per context using "_role_`@`_context_". You can also assign a role for context "`*`", which means that the user gets that role for any context existing now, or created in the future.

### Assigning roles

In Axon Server EE we can grant specific roles to applications and users that will allow specific operations. Apart from just assigning the roles, you must also indicate for which context the role is granted, so that an application/user that has rights on only one context is not able to access data from other contexts.‌

A summary of the various roles is depicted below

| Role Name | Description |
| :--- | :--- |
| ADMIN | Administer the cluster, manage contexts, users and applications |
| VIEW\_CONFIGURATION | View cluster configuration with contexts, users, applications, replication groups, and plugins (only for \_admin context) | 
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

### Accounts without a password

Using the CLI it is possible to create an account without a password. This does not mean that the account requires no password to login, but rather that the account is only used to assign roles to, while the password needs to be checked using an external tool. To create such an account, use the "`--no-password`" option:

```text
$ java -jar axonserver-cli.jar register-user -u username -r roles --no-password
```

See the section on OAuth2 integration for an example.