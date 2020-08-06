#Cluster template

With the Cluster Template feature we enabled users to maintain cluster configuration as a code.

Cluster template is defined as a YAML file which describes cluster configuration. It is possible to pre-define replication groups, contexts, metadata, applications (with tokens), users and their roles and that configuration can be shared across teams.

{% hint style="warn" %}
Cluster template runs exactly once, on the first clean Axon Server start-up, if there is no previous cluster configuration defined. Therefore, cluster template WILL NOT override any exciting configuration, and its purpose is to be used during active development and to share cluster configuration across the developing team.
{% endhint %}

{% hint style="warn" %}
As clustering is an Axon Server EE feature, license needs to be provided before using this feature.
{% endhint %}


## Usage

To use Cluster template all you need to do is define a valid cluster template yml file. If this file is present on a clean Axon Server startup, it will automatically be picked up and the cluster will be configured accordingly.

{% hint style="warn" %}
Each cluster node needs to have the cluster template yml copy . Each node will read this file, find its own configuration within configuration and configure itself.
{% endhint %}

Default path from which Axon Server reads configuration is `./cluster-template.yml`

You can override this path anytime by setting Axon Server property: 
`axoniq.axonserver.clustertemplate.path:/mypath/cluster-template.yml`

### Configuration

Below you can find an example of basic cluster setup: _admin & default context in separate replication nodes, replicated across all nodes that are marked as primary.

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

In order to avoid mistakes while working with cluster templates we implemented an export button that will generate cluster template files based on current cluster configuration. 

![Cluster Template export button location](/.gitbook/assets/cluster-template-export-button.png)
_Location of export button at Settings page_

**Recommended way for creating advanced cluster setup is to**

* Start a fresh Axon Server (use basic cluster template setup mentioned above)
* Configure a cluster via UI by creating users, applications, replication groups & contexts
* Use export button located at Settings tab > Configuration panel  to download current cluster configuration
* Replace basic cluster template with newly exported cluster template configuration


{% hint style="warn" %}
Use export button from main - leader node.
{% endhint %}

