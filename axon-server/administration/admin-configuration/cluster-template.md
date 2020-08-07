#Cluster template

With the Cluster Template feature we enable users to maintain cluster configuration as code.

The cluster template is defined as a YAML file, describing a cluster's configuration. It is possible to predefine replication groups, contexts, metadata, applications (with tokens), users and their roles, so that the configuration can be shared across teams.

{% hint style="warn" %}
The cluster template runs exactly once, on the first clean Axon Server start-up, if there is no previous cluster configuration defined. Therefore, the cluster template **will not** override any existing configuration. Its purpose is to be used during active development, to be able to share the configuration across development teams.
{% endhint %}

{% hint style="warn" %}
As clustering is an Axon Server EE feature, a license needs to be provided to be able to use this feature.
{% endhint %}


## Usage

To use the cluster template feature, all you need to do is define a valid cluster template yml file. If this file is present on a fresh Axon Server startup, it will automatically be picked up and the cluster will be configured accordingly.

{% hint style="warn" %}
Each cluster node needs to have the cluster template yml file copy. Each node will read this file, find its own configuration and configure itself.
{% endhint %}

Default path from which Axon Server reads configuration is `./cluster-template.yml`

You can override this path anytime by setting Axon Server property: 
`axoniq.axonserver.clustertemplate.path:/mypath/cluster-template.yml`

### Configuration

Below you can find an example of a basic cluster setup: the _admin and default contexts are in separate replication nodes, replicated across all nodes that are marked as primary.

```yaml
axoniq:
  axonserver:
    cluster-template:
      first: axonserver-enterprise-1
      replicationsGroups:
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

**Recommended way for creating an advanced cluster setup**

* Start a fresh Axon Server setup (use basic cluster template setup mentioned above).
* Configure a cluster via the UI, by creating users, applications, replication groups and contexts.
* Use the export button located at "Settings -> Configuration" panel  to download the current cluster configuration.
* Replace the basic cluster template with the newly exported cluster template configuration.


{% hint style="warn" %}
Use export button from main - leader node.
Leader node always contains full cluster configuration, while it might happen that followers don't.
{% endhint %}
