# Clusters

> Note - This feature is only available in Axon Server Enterprise

## Introduction

Axon Server Enterprise Edition can be deployed as a _**cluster**_ to guarantee high availability. A cluster of Axon Server EE nodes will provide multiple connection points for \(Axon Framework-based\) client applications, and thus share the load of managing message delivery and event storage. Client applications will dynamically connect to a node in the cluster and automatically reconnect to another, should the node that they are currently connected to become unreachable.‌

An Axon Server EE cluster has 3 main areas of administration,

* [Cluster Nodes](clustering.md#cluster-nodes) - "Instances" of Axon Server EE that need to be part of the EE cluster.
* [Replication Groups](replication-groups.md) - Responsible for event data replication and transaction management between the various nodes of a cluster.
* [Contexts](multi-context.md) - Responsible for event storage within the various nodes of a cluster.

A visual representation of the relationship between the 3 is shown below.

![Relationship between Cluster Nodes / Replication Groups and Contexts](../../.gitbook/assets/clusters.jpg)

## Setup Process

The cluster setup process always begins by designating any one clean/uninitialized Axon Server EE node as the first member of the cluster. You can then run the "[init-cluster](admin-configuration/command-line-interface.md#cluster-enterprise-edition-only)" command on it which will create the following replication groups and contexts -&gt; _admin/default._

From thereon, there are multiple ways to continue the setup depending upon your Event Store deployment topology. 

* Any other Axon Server EE node can be added to the cluster using the "register-node" command without associating it with any Replication Group / Context.
* New Replication Groups/Contexts can be added and cluster member nodes can be associated with these. 
* Member nodes can be removed from the cluster at any point of time.

Axon provides two ways for automating cluster configuration. The first is the [Automatic Initialization](clustering.md#automatic-initialization) feature and the other is the [Cluster Template](clustering.md#cluster-templates) feature.

### Automatic initialization

The manual process of member registration of the cluster can be bypassed by setting a couple of properties in the axonserver.properties file.

axoniq.axonserver.autocluster.first=internal-hostname:internal-port axoniq.axonserver.autocluster.contexts=context1,context2

The _axoniq.axonserver.autocluster.first_ property defines the first node in the cluster, by specifying its internal hostname \(the hostname used by other Axon Server nodes to connect to this host\), and the internal port. If the internal port is default \(8224\) it can be omitted.‌

_axoniq.axonserver.autocluster.contexts_ defines the contexts to create on the first node and the context to join for the other nodes. All of these contexts will be joined as primary nodes. When you don't specify any contexts, the initial node will only create an admin context, the other nodes will join the cluster, but not be a member of any contexts.‌

The autocluster properties will only take effect on a clean start of a node. If a node is already initialized, it will not create any contexts anymore, nor join the cluster again.‌

### Cluster Templates

> Note - This feature is only available in Axon Server Enterprise \(since v4.4\)

The cluster template is defined as a YAML file, describing a cluster's configuration. It is possible to predefine replication groups, contexts, metadata, applications \(with tokens\), users and their roles, so that the configuration can be shared across teams.

{% hint style="info" %}
The cluster template runs exactly once, on the first clean Axon Server start-up, if there is no previous cluster configuration defined. Therefore, the cluster template **will not** override any existing configuration. Its purpose is to be used during active development, to be able to share the configuration across development teams.
{% endhint %}

#### Usage

To use the cluster template feature, all you need to do is define a valid cluster template yml file. If this file is present on a fresh Axon Server startup, it will automatically be picked up and the cluster will be configured accordingly.

{% hint style="info" %}
Each cluster node needs to have the cluster template yml file copy. Each node will read this file, find its own configuration and configure itself.
{% endhint %}

Default path from which Axon Server reads configuration is `./cluster-template.yml`

You can override this path anytime by setting Axon Server property: `axoniq.axonserver.clustertemplate.path:/mypath/cluster-template.yml`

#### Configuration

Below you can find an example of a basic cluster setup: the \_admin and default contexts are in separate replication nodes, replicated across all nodes that are marked as primary.

```yaml
axoniq:
  axonserver:
    cluster-template:
      first: internal-hostname:internal-port
      replicationGroups:
      - name: _admin
        roles:
        - node: axonserver-enterprise-1
          role: PRIMARY
        - node: axonserver-enterprise-2
          role: PRIMARY
        - node: axonserver-enterprise-3
          role: PRIMARY
        contexts:
        - name: _admin
      - name: default
        roles:
        - node: axonserver-enterprise-2
          role: PRIMARY
        - node: axonserver-enterprise-3
          role: PRIMARY
        - node: axonserver-enterprise-1
          role: PRIMARY
        contexts:
        - name: default
      applications: []
      users: []
```

![Cluster overview after default configuration is applied](../../.gitbook/assets/cluster-template-default-configuration.png)

_Cluster overview after default configuration is applied_

#### Export/Generator

In order to avoid mistakes while writing a cluster configuration file, we have implemented an export button that will generate a cluster template file based on current setup.

![Cluster Template export button location](../../.gitbook/assets/cluster-template-export-button.png)

_Location of export button at Settings page_

**Recommended mechanism - Creating an advanced cluster setup**

* Start a fresh Axon Server setup \(use basic cluster template setup mentioned above\).
* Configure a cluster via the UI, by creating users, applications, replication groups and contexts.
* Use the export button located at "Settings -&gt; Configuration" panel  to download the current cluster configuration.
* Replace the basic cluster template with the newly exported cluster template configuration.

{% hint style="info" %}
Use export button from any admin node to ensure that the configuration file contains all the relevant information.
{% endhint %}

