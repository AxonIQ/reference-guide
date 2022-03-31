# Flow Control

Flow control is the process of managing the rate of data transmission between two nodes to prevent a fast sender from overwhelming a slow receiver.

In the messaging platform flow control is possible both between the messaging platform and the message handlers, and between the nodes in the messaging platform cluster.

_Messaging Platform - Messaging Handler:_

The client \(i.e. the Axon Application\) needs to set the following properties to configure flow control:

* `axon.axonserver.initial-nr-of-permits` \[1000\] - number of messages that the server can initially send to client.
* `axon.axonserver.nr-of-new-permits` \[500\] - additional number of messages that the server can send to client.
* `axon.axonserver.new-permits-threshold` \[500\] -  when client reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

_Axon Server Nodes:_

Set the following properties to control 'list aggregate events' prefetch rate, per aggregate and per segment:

* `axoniq.axonserver.event.aggregate.prefetch` \[5\] - Ensures that backpressure signals from clients are split into batches. The initial request amount is {prefetch}*5, and subsequent (or replenishing) request amount is {prefetch}
* `axoniq.axonserver.event.events-per-segment-prefetch` \[10\] - the maximum prefetched events from each opened event segment (max two opened segments in parallel)


Set the following properties to set flow control on the synchronization between nodes in an Axon Server cluster:

* `axoniq.axonserver.commandFlowControl.initial-nr-of-permits` \[10000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.commandFlowControl.nr-of-new-permits` \[5000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.commandFlowControl.new-permits-threshold` \[5000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.
* `axoniq.axonserver.queryFlowControl.initial-nr-of-permits` \[10000\] - number of messages that the master can initially send to replica.
* `axoniq.axonserver.queryFlowControl.nr-of-new-permits` \[5000\] - additional number of messages that the master can send to replica.
* `axoniq.axonserver.queryFlowControl.new-permits-threshold` \[5000\] - when replica reaches this threshold in remaining messages, it sends a request with additional number of messages to receive.

## Streaming query

Flow control and stream cancellation features are only available with Axon Server 4.6.0 and up. 
When streaming queries are used with Axon Server versions before 4.6.0, it will work although without the following essential features.
Under the hood, backpressure does `Hop to Hop` signal propagation (see below) and inherits gRPC's [HTTP2-based backpressure model](https://developers.google.com/web/fundamentals/performance/http2/#flow_control).

As a result, backpressure will not behave intuitively and will not propagate exact request signals from consumer to producer.
HTTP/2 and Netty flow control have internal buffers based on message size. 
In turn, Axon Framework and Axon Server prefetch messages into internal buffers based on message count.
The result is that the producer will send a number of messages until it fills all the buffers.
Only then will backpressure kick in.

> **Hop to hop**
>
> The backpressure signal is propagated per-hop.
> This approach makes it not an end-to-end connection that allows intermediate Axon Server instances to handle backpressure between two connections and pre-fetch additional messages to increase overall performance.

It's important to note that similar to backpressure, the cancellation signal is also per hop.
This means it's propagated over the network to Axon Server and then to the producer.
This solution will thus introduce some latency in the stream cancellation.
Even though there is potential latency involved in cancellation, any messages produced **after** the consumer signaled cancellation will be ignored.


