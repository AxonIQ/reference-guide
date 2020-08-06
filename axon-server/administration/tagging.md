# Tagging

You can use tagging in Axon for 2 different scenarios:

1. You want to control which command handler or query handler executes a command or query, or
2. You want to control to which Axon Server node a particular client connects.

## Tag based routing

> **Note**
>
> This feature is only available with Axon Server version 4.4 onwards.
>
Axon Server matches metadata elements provided in a command or a query to tags defined on the command and client handling 
applications. For instance, you could tag commands with a country code and set the country code in the tags of the handler application.
In this case, Axon Server will try to find a command handler with the same country code and send the command to that handler.
If there are multiple applications found with a command handler for the requested command and matching tags, Axon Server will
pick one based on the provided routing key. The same will happen if there are no applications found with matching tags, so 
commands will not fail because there are no matching tags. 
You can, of course, have multiple tags on an application, Axon Server will select the one with most the matching tags. 

## Connection distribution 

> **Note**
>
> This feature is only available in Axon Server Enterprise and compatible with version 4.2 onwards of Axon.

To optimize the connections between framework clients and Axon Server, it is possible to tag both clients and Axon Server with information which describes properties of those clients and nodes. Axon Server will then use these tags to make a connection to the client.‌

A typical tag could describe where the respective client and server node are running \(e.g. a cloud provider's region\) encouraging that the nearest, and therefore fastest, connections are made.‌

## Tag Matching <a id="tag-matching"></a>

A match is made when the the label and the value of the tag matches. For example, a server tagged with `computeRegion=europe` will match with a client tag of `computeRegion=europe`. Similarly if a server is tagged with `computeRegion=europe` and client is tagged with `computeRegion=asia` then this is not considered a match.‌

If both client and server are tagged with multiple tags then the connection that is made will be based on the highest number of matches. If multiple nodes have a equal number of matching tags then the node with the lightest load is chosen.‌

If no matching tags are available at all, it will connect to any available node.

## Enabling tagging <a id="enabling-tagging"></a>

To enable tagging you must configure both the client and server.

### Axon Server <a id="axon-server"></a>

You can configure the tags that you would like for each node through properties using `axoniq.axonserver.tags.computeRegion=europe`. Additional tags can be specified by adding to the `tags` element \(e.g. `axoniq.axonserver.tags.networkSpeed=fast`\).‌

### Client <a id="client"></a>

On the client side, there are two approaches through which you can specify tags. Either the properties file is expanded \(similar to server configuration, also allowing additional tags to be added\) in a Spring Boot environment, or a `TagsConfiguration` object is registered to the `Configurer`.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Map<String, String> tags = new HashMap();
// Insert tags
DefaultConfigurer.defaultConfiguration()
    .configureTags(config -> new TagsConfiguration(tags));
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```text
axon.tags.computeRegion=europe
```
{% endtab %}
{% endtabs %}

