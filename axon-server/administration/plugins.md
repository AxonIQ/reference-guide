# Axon Server Plugins
> **Note**
>
> This feature is only available with Axon Server version 4.5 onwards.
>
Plugins allow you to add specific interceptors in Axon Server. This enables intercepting commands and queries before they are updated and after they are executed, and it enables intercepting events and snapshots before they are stored and when they are read.

A plugin is an OSGi package, that you can upload to Axon Server. The package can implement a number of interceptors, Axon Server will discover these and start using them. The OSGi package runs in an OSGi container, using its own class loader and it only has access to specific Axon Server classes.

A plugin package has a symbolic name and version. This information is part of the information in the manifest file of the package. If you upload a package with the same name and version, it will replace the already available package.
You can have multiple versions of the same package available.

Plugins are managed at a context level, you can configure and activate plugins per context. Per context, you can only have one version of a plugin active. In Axon Server Standard Edition there only is one context, you can still active or pause the plugin for the default context.

Axon Server stores plugins on disk on each node. It is important that the location where the plugins are stored is persistent. By default, Axon Server stores the plugin in the directory plugins/bundles. Apart from this directory, there is also a cache directory used by the OSGi container internally (plugins/cache by default). The cache directory does not need to be persistent, Axon Server reinstalls the configured extensions on restart.

## Plugin administration
You can administer plugins through the UI or using the command-line interface.

### Upload plugin

This uploads aplugin to Axon Server and makes it available for further configuration.
When you upload the plugin Axon Server will perform some basic validation, to verify that it is a valid OSGi bundle and it can be loaded in the OSGi container.
If the uploaded plugin has the same name and version as an already existing plugin, it will immediately replace the existing plugin.

To upload an plugin through the command-line interface, use:
```bash
java -jar axonserver-cli.jar upload-plugin -f [file] 
```
### Configuring a plugin
If your plugin contains registered services of the type _io.axoniq.axonserver.plugin.ConfigurationListener_, you can configure the plugin through Axon Server. In the UI you will see a form with the properties that are defined in the ConfigurationListeners, initially filled with the default values. Note that you set the values per context.

A plugin can have multiple ConfigurationListeners, each one is identified by a name. This name is shown in the Axon Server UI.

If you want to set the configuration values through the command-line interface, you have two options, either specify the values as parameters in the command or provide a YAML file containing the parameter values.

Here’s an example of a configuration listener that defines two properties:

```java
package org.sample;

import io.axoniq.axonserver.plugin.Configuration;
import io.axoniq.axonserver.plugin.ConfigurationListener;
import io.axoniq.axonserver.plugin.PluginPropertyDefinition;

import java.util.Map;

import static java.util.Arrays.asList;

/**
 * @author Marc Gathier
 */
public class SampleConfigurationListener implements ConfigurationListener {

    private final Configuration configuration  = new Configuration(
            asList(
                    PluginPropertyDefinition.newBuilder("mypropid1", "My first property").build(),
                    PluginPropertyDefinition.newBuilder("mypropid2", "My second property").build()
            ),
            "myname"

    );

    @Override
    public Configuration configuration() {
        return configuration;
    }

    @Override
    public void updated(String s, Map<String, ?> map) {

    }

}
```

To set the properties through the command line using the parameters option use:
```bash
java -jar axonserver-cli.jar upload-plugin -p <plugin> -v <version> -c <context> -prop myname:mypropid1=myvalue -prop myname:mypropid2=myvalue2
```

To do the same using a YAML file create a file with the following content:
```yaml
myname:
  mypropid1: myvalue
  mypropid2: myvalue3
```

And use the following command-line command:
```bash
java -jar axonserver-cli.jar configure-plugin -p <plugin> -v <version> -c <context> -f <filename>
```
### Activating a plugin

You can activate a plugin through the UI or use the following command-line command:
```bash
java -jar axonserver-cli.jar activate-plugin -p <plugin> -v <version> -c <context> 
```

### Pausing a plugin

If you want to stop an plugin being used temporarily for a context, you can pause it through the UI or the 
command-line interface. This will keep all the configuration for the plugin, so when you later start it again
it will use the same configuration as before. 

The command-line command for this is:
```bash
java -jar axonserver-cli.jar pause-plugin -p <plugin> -v <version> -c <context> 
```

### Unregistering a plugin

This unregisters a plugin for a specific context. The plugin will still be active for other contexts.  

The command-line command for this is:
```bash
java -jar axonserver-cli.jar unregister-plugin -p <plugin> -v <version> -c <context> 
```

### Deleting a plugin

Deleting a plugin unregisters it from all the contexts and deletes the package from all the nodes.

The command-line command for this is:
```bash
java -jar axonserver-cli.jar delete-plugin -p <plugin> -v <version> 
```











