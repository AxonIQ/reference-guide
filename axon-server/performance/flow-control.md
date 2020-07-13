# Flow Control

Flow control is the process of managing the rate of data transmission between two nodes to prevent a fast sender from overwhelming a slow receiver.

In the messaging platform flow control is possible both between the messaging platform and the message handlers, and between the nodes in the messaging platform cluster.

_Messaging Platform - Messaging Handler:_

The client \(i.e. the Axon Application\) needs to set the following properties to configure flow control:

* `axon.axonserver.initial-nr-of-permits` \[1000\] - number of messages that the server can initially send to client.
* `axon.axonserver.nr-of-new-permits` \[500\] - additional number of messages that the server can send to client.
* `axon.axonserver.new-permits-threshold` \[500\] -  when client reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

_Axon Server Nodes:_

Set the following properties to set flow control on the synchronization between nodes in an Axon Server cluster:

* `axoniq.axonserver.commandFlowControl.initial-nr-of-permits` \[10000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.commandFlowControl.nr-of-new-permits` \[5000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.commandFlowControl.new-permits-threshold` \[5000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.
* `axoniq.axonserver.queryFlowControl.initial-nr-of-permits` \[10000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.queryFlowControl.nr-of-new-permits` \[5000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.queryFlowControl.new-permits-threshold` \[5000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

