# Multi-context

> Note: This feature is only available on the Enterprise Edition of AxonServer

You can use a single AxonServer \(cluster\) to store events for multiple bounded contexts. Each context will have its own set of files \(stored in a separate directory\). Each context may have a different master in an AxonServer cluster.

Creating a new context is done using the user interface, or via the command line interface:

```bash
$ java -jar ./axonserver-cli.jar create-context -S http://[node]:[port] -c [context-name] -n [members]
```

The server address here is the address of the leader for the _\_admin_ context. It is not possible to delete the _\_admin_ context. Members is a comma separated list of node names that should be members of the new context.

The context name must match the following regular expression "\[a-zA-Z\]\[a-zA-Z\_-0-9\]\*", so it should start with a letter \(uppercase or lowercase\), followed by a combination of letters, digits, hyphens and underscores.

