# Cluster Templates

> Note - This feature is only available in Axon Server Enterprise

Cluster Templates enable administrators to maintain AxonServer [cluster](../clustering.md) configuration as code.

The cluster template is defined as a YAML file, describing a cluster's configuration. It is possible to predefine replication groups, contexts, metadata, applications \(with tokens\), users and their roles, so that the configuration can be shared across teams.

{% hint style="info" %}
The cluster template runs exactly once, on the first clean Axon Server start-up, if there is no previous cluster configuration defined. Therefore, the cluster template **will not** override any existing configuration. Its purpose is to be used during active development, to be able to share the configuration across development teams.
{% endhint %}

## Usage

To use the cluster template feature, all you need to do is define a valid cluster template yml file. If this file is present on a fresh Axon Server startup, it will automatically be picked up and the cluster will be configured accordingly.

{% hint style="info" %}
Each cluster node needs to have the cluster template yml file copy. Each node will read this file, find its own configuration and configure itself.
{% endhint %}

Default path from which Axon Server reads configuration is `./cluster-template.yml`

You can override this path anytime by setting Axon Server property: `axoniq.axonserver.clustertemplate.path:/mypath/cluster-template.yml`

### Configuration

Below you can find an example of a basic cluster setup: the \_admin and default contexts are in separate replication nodes, replicated across all nodes that are marked as primary.

```yaml
axoniq:
  axonserver:
    cluster-template:
      first: axonserver-enterprise-1
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

![Cluster overview after default configuration is applied](/.gitbook/assets/cluster-template-default-configuration.png) 

_Cluster overview after default configuration is applied_

### Export/Generator

In order to avoid mistakes while writing a cluster configuration file, we have implemented an export button that will generate a cluster template file based on current setup.

![Cluster Template export button location](/.gitbook/assets/cluster-template-export-button.png) 

_Location of export button at Settings page_

**Recommended mechanism - Creating an advanced cluster setup**

* Start a fresh Axon Server setup \(use basic cluster template setup mentioned above\).
* Configure a cluster via the UI, by creating users, applications, replication groups and contexts.
* Use the export button located at "Settings -&gt; Configuration" panel  to download the current cluster configuration.
* Replace the basic cluster template with the newly exported cluster template configuration.

{% hint style="info" %}
Use export button from main - leader node. Leader node always contains complete cluster configuration, while it might happen that followers don't.
{% endhint %}

