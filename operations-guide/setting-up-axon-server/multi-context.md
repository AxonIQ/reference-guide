# Multi-Context

 > NOTE: This feature is only available on the Enterprise Edition of AxonServer

You can use a single AxonServer (cluster) to store events for multiple bounded contexts. Each context will have its own
set of files (stored in a separate directory). Each context may have a different master in an AxonServer cluster.

Creating a new context is done using the user interface, or via the command line interface:

```bash
./axonserver-cli.jar create-context -S http://axonserver-node:8024 -c context-name
```

The server address here is the address of the master for the *default* context. It is not possible to delete the *default*
context.
