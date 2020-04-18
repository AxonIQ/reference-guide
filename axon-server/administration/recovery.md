# Recovery

> **Note**
>
> This feature is only available in Axon Server Enterprise version 4.3 onwards.

In case of a major network change, when the host names of all Axon Server nodes have changed, or when you want to move an Axon Server cluster to a new environment, you will have to start Axon Server in recovery mode. This is needed as Axon Server maintains the information about the nodes in the cluster in its control database, and this needs to be updated on all nodes.

Axon Server will start in recovery mode when it finds a file called _recovery.json_ in its startup directory, or in a location defined by the property _axoniq.axonserver.recoveryfile_. The recovery file must contain a JSON array of the nodes in the cluster with their host names and port numbers, for example:

```javascript
[
  {
   "name": "axonserver-1",
   "hostName": "axonserver-1",
   "internalHostName": "axonserver-1.internal",
   "grpcInternalPort": 8224,
   "httpPort": 8024,
   "grpcPort": 8124
  },
  {
   "name": "axonserver-2",
   "hostName": "axonserver-2",
   "internalHostName": "axonserver-2.internal",
   "grpcInternalPort": 8224,
   "httpPort": 8024,
   "grpcPort": 8124
  },
  {
   "name": "axonserver-3",
   "hostName": "axonserver-3",
   "internalHostName": "axonserver-3.internal",
   "grpcInternalPort": 8224,
   "httpPort": 8024,
   "grpcPort": 8124
  }
]
```

The _hostName_ defines the hostname that clients will use to connect to the Axon Server node. The _internalHostName_ contains the name one Axon Server uses to connect to the other. Any elements that have not changed \(except for the name\) may be omitted, so if you want to use the same port numbers you can omit the _grpcInternalPort_, _httpPort_ and _grpcPort_ elements.

In the sample above, the node names for the Axon Server nodes will remain the same as they were before. As Axon Server derives its node name from the host name, when not explicitly set in properties, this may cause an issue when host names are changed. To change the node names for Axon Server nodes, add an _oldName_ element in the JSON file.

```javascript
[
  {
   "name": "new-axonserver-1",
   "oldName": "axonserver-1",
   "hostName": "new-axonserver-1",
   "internalHostName":" new-axonserver-1.internal"
  },
  {
   "name": "new-axonserver-2",
   "oldName": "axonserver-2",
   "hostName": "new-axonserver-2",
   "internalHostName": "new-axonserver-2.internal"
  }, 
  {
   "name": "new-axonserver-3",
   "oldName": "axonserver-3",
   "hostName": "new-axonserver-3",
   "internalHostName": "new-axonserver-3.internal"
  }
]
```

