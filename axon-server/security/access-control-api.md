# Access Control on the REST and gRPC APIs

When access control for Axon Server is enabled, you'll need to provide user credentials or a token. The UI uses username and password to establish a session, while the [CLI can use a token](access-control-cli.md). If you are using tools such as "`curl`", or have coded your own tool to access the REST API, you'll need to do this also. The gRPC APIs do _not_ accept user credentials, but always expect a token. Note that the internal API (by default on port 8224) will only accept the "internal-token", while the client API (by default on port 8124) will only accept application tokens. 

## Passing User Credentials to the REST API

Axon Server expects user credentials using the Basic Authentication method. For example, to list the contexts with "`curl`" using user account "user" with password "password", you would use:

```
$ curl -u user:password http://localhost:8024/v1/public/context | jq
[
  {
    "changePending": false,
    "leader": null,
    "pendingSince": 0,
    "metaData": {
      "event.index-format": "JUMP_SKIP_INDEX",
      "snapshot.index-format": "JUMP_SKIP_INDEX"
    },
    "roles": [
      {
        "role": "PRIMARY",
        "node": "node-1"
      },
      {
        "role": "PRIMARY",
        "node": "node-2"
      },
      {
        "role": "PRIMARY",
        "node": "node-3"
      }
    ],
    "replicationGroup": "_admin",
    "context": "_admin"
  },
  {
    "changePending": false,
    "leader": null,
    "pendingSince": 0,
    "metaData": {
      "event.index-format": "JUMP_SKIP_INDEX",
      "snapshot.index-format": "JUMP_SKIP_INDEX"
    },
    "roles": [
      {
        "role": "PRIMARY",
        "node": "node-1"
      },
      {
        "role": "PRIMARY",
        "node": "node-2"
      },
      {
        "role": "PRIMARY",
        "node": "node-3"
      }
    ],
    "replicationGroup": "default",
    "context": "default"
  }
]
```

## Passing a Token to the REST and gRPC APIs

To pass a token to the REST and gRPC APIs, you must add the "`AxonIQ-Access-Token`" header, with the token as value. For example, when using "`curl`" as in the previous section, but now with a token instead of user credentials:

```
$ curl -H 'AxonIQ-Access-Token: cfd7304a-950e-4e32-86ba-5ecb2c4d23ec' https://localhost:8024/v1/public/context | jq
...output omitted
```
