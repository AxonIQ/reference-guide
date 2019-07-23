# Configuring Client Connections (Tagging)

> Note - This feature is only available in Axon Server Enterprise

To optimise the connections between framework clients and Axon Server, its is possible to tag both clients and Axon Server with information which describes properties of those clients and nodes. Axon Server will then use these tags to make a connection to the client.

A typical tag could describe where the respective client and server node are running (e.g. a cloud provider's region) encouraging that the nearest, and therefore fastest, connections are made.

## Tag Matching

A match is made when the the label and the value of the tag matches. For example, a server tagged with `"computeRegion": "europe"` will match with a client tag of `"computeRegion": "europe"`. Similarly if server is tagged with `"computeRegion": "europe"` and client is tagged with `"computeRegion": "asia"` then this is not considered a match.

If both client and server are tagged with multiple tags then the connection that is made will be based on the highest number of matches. If multiple nodes have a equal number of matching tags then the node with the lightest load is chosen.

## Enabling tagging

To enable tagging you must modify configuration on both the client and the server.

### Axon Server

Set the value of property `axoniq.axonserver.clients-connection-strategy` to `matchingTags` to enable the tagging connection strategy.

Then configure the tags that you would like for each node through properties using `axoniq.axonserver.tags.computeRegion: europe`. Additional tags can be specified by adding to the `tags` element (e.g. `axoniq.axonserver.tags.networkSpeed: fast`).

### Client

On the client side, you can add tags by using the property `axon.tags.computeRegion: europe` and, similar to server configuration, additional tags can be added.
