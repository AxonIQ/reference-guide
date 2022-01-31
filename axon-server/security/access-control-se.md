# Access Control for Axon Server SE

When you set the "`axoniq.axonserver.accesscontrol.enabled`" property to "`true`", Axon Server will require a token or user account for access to its APIs.

## Tokens in Axon Server SE

In Axon Server SE, there are two tokens that can be defined:

| Property                                | Purpose |
|:----------------------------------------| :--- |
| `axoniq.axonserver.accesscontrol.token` | Define a token with normal (limited) rights. |
| `axoniq.axonserver.accesscontrol.adminToken`          | Define a token with administrative rights. Can also be specified as "`axoniq.axonserver.admin-token`". |

Generally, you will use the admin-token only for the CLI, to issue commands for managing user accounts and plugins. Axon Framework based applications should only need the non-admin token. If you whish to use tools to access the REST API directly, you must add an HTTP header named "`AxonIQ-Access-Token`", as in the following example:

```bash
$ curl -H 'AxonIQ-Access-Token: my-token' -s http://localhost:8024/v1/public/users | jq
[
  {
    "userName": "admin",
    "password": null,
    "roles": [
      "ADMIN@*"
    ]
  }
]
```

## User Accounts in Axon Server SE

When you create a user account, you can optionally assign the role "`ADMIN`", which is a shorthand for "`ADMIN@*`" and will allow the user to access the user-administration and plugin pages in the UI. These user accounts are generally only used for the UI, although they are also valid for access to the REST API, using Basic Authentication.
