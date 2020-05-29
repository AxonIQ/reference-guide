# Configuration

There are several configuration options for the Axon Server that can be done to streamline and optimize your Axon Server SE/EE deployment. 

The configurations can be maintained/supplied in three ways.

* [Configuration File](configuration.md#configuration-file)
* [Command Line](configuration.md#command-line)
* [Environment Variables](configuration.md#environment-variable-s)

### _Configuration File_

The most commonly and preferred way is to have an _axonserver.properties_ or __ _axonserver.yml_  ****file which contains the desired configuration parameters. The location of the file should be the current working directory or alternatively can be placed within a "_config"_ subdirectory \(relative to the current working directory\). 

An important note - In case both files are detected by Axon Server, it will read from both.

### _Command-Line_

In case the server is being started using “java –jar ...”, we can also supply individual configuration properties with _“-Dproperty=value”_

### _Environment Variable\(s\)_

Configuration values can also be supplied using environment variables. The parameter name should be all in upper case with any kebab-case\(-\) / camelCase and snake\_case\(\_\) substituted with _"\_"_

### _Recommendations_

There are some recommendations around Axon Server EE/SE configuration,

* Use “./axonserver.properties” for common settings 
* Use “./config/axonserver.properties” for environment/node-specific overrides. 
* Use “-D” or environment variables for one-time settings

## Configuration Properties

A list of all the configuration properties by area is denoted below. Unless explicitly specified all property names are to be prefixed with "_axoniq.axonserver_"

<table>
  <thead>
    <tr>
      <th style="text-align:left">Area</th>
      <th style="text-align:left">Property Name</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Default Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><em><b>Names / Ports</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">name</td>
      <td style="text-align:left">Unique node name of the Axon Server node</td>
      <td style="text-align:left">Hostname of the server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">hostname</td>
      <td style="text-align:left">Hostname of the Axon Server node as communicated to clients</td>
      <td style="text-align:left">Hostname of the server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">internal-hostname</td>
      <td style="text-align:left">
        <p>Hostname as communicated to other nodes</p>
        <p>of the cluster</p>
      </td>
      <td style="text-align:left">Hostname of the server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">domain</td>
      <td style="text-align:left">
        <p>Domain of this node as communicated to clients. Optional, if set will
          be appended</p>
        <p>to the hostname in communication with clients</p>
      </td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">port</td>
      <td style="text-align:left">gRPC port for the Axon Server node</td>
      <td style="text-align:left">8124</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">server.port <em>(no prefix)</em>
      </td>
      <td style="text-align:left">HTTP port for the Axon Server console</td>
      <td style="text-align:left">8024</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">internal.port</td>
      <td style="text-align:left">
        <p>gRPC port for communication between Axon Server nodes within a cluster</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
      <td style="text-align:left">8224</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">tags</td>
      <td style="text-align:left">Key/value pairs of tags for tag based connections for clients.</td>
      <td
      style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">devmode.enabled</td>
      <td style="text-align:left">Development mode which allows deleting all events from event store <em>(Axon SE only)</em>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">set-web-socket-allowed-origins</td>
      <td style="text-align:left">Set WebSocket CORS Allowed Origins</td>
      <td style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Data</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.storage</td>
      <td style="text-align:left">Path where (regular) events are stored as segmented files on disk</td>
      <td
      style="text-align:left">./data directory</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.storage</td>
      <td style="text-align:left">Path where Snapshot Events are stored as segmented files on disk</td>
      <td
      style="text-align:left">./data directory</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">controldb-path</td>
      <td style="text-align:left">Path where Axon Server&apos;s control database (axonserver-controldb)
        is createed</td>
      <td style="text-align:left">./data directory</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Logging</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>logging.level.{package_name}</p>
        <p><em>(no prefix)</em>
        </p>
      </td>
      <td style="text-align:left">
        <p>Change the logging level for specific packages or classes.</p>
        <p></p>
        <p><em>(e.g. logging.level.io.axoniq.axonserver = INFO)</em>
        </p>
      </td>
      <td style="text-align:left">WARN level for all packages</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">logging.file <em>(no prefix)</em>
      </td>
      <td style="text-align:left">File name where log entries should be written to <em>(e.g. logging.file = messaging.log)</em>
      </td>
      <td style="text-align:left">stdout</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">logging.path <em>(no prefix)</em>
      </td>
      <td style="text-align:left">
        <p>Location where log files should be created</p>
        <p></p>
        <p><em>(e.g. logging.path = /var/log)</em>
        </p>
      </td>
      <td style="text-align:left">stdout</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Auto-Clustering</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">autocluster.first</td>
      <td style="text-align:left">For auto cluster option, set to the internal host name for the first node
        in the cluster</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">autocluster.contexts</td>
      <td style="text-align:left">For auto cluster option, defines the list of contexts to connect to or
        create</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>SSL (Axon Server - HTTP Port)</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>security.require-ssl</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Determines whether the server has ssl enabled on the HTTP port</td>
      <td
      style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>server.ssl.key-store-type</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Keystore type (should be PKCS12)</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>server.ssl.key-store</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Location of the keystore</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>server.ssl.key-store-password</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Password to access the keystore</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>server.ssl.key-alias</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Alias to be used to access the keystore</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>SSL (Axon Server - gRPC Port)</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.enabled</td>
      <td style="text-align:left">Determines whether the server has ssl enabled on the gRPC port</td>
      <td
      style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.cert-chain-file</td>
      <td style="text-align:left">Location of the public certificate file</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.private-key-file</td>
      <td style="text-align:left">Location of the private key file</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.internal-chain-file</td>
      <td style="text-align:left">
        <p>Location of the certificate file used for cluster-internal traffic</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">internal-trust-manager-file</td>
      <td style="text-align:left">
        <p>Location of the keystore that certifies other certificates</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Messaging (Between Clients and Axon Server)</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">max-message-size</td>
      <td style="text-align:left">Maximum size of a message to be sent to the node</td>
      <td style="text-align:left">4 MB</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">initial-nr-of-permits</td>
      <td style="text-align:left">Number of messages that the server can initially send to a client</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">nr-of-new-permits</td>
      <td style="text-align:left">Additional number of messages that the server can send to a client</td>
      <td
      style="text-align:left">500</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">new-permits-threshold</td>
      <td style="text-align:left">When a client reaches this threshold in remaining messages, it sends a
        request with additional number of messages to receive</td>
      <td style="text-align:left">500</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Messaging (Between nodes of an Axon Server Cluster)</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">commandFlowControl.initial-nr-of-permits</td>
      <td style="text-align:left">Number of command messages that the master can initially send to a replica</td>
      <td
      style="text-align:left">10000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">commandFlowControl.nr-of-new-permits</td>
      <td style="text-align:left">Additional number of command messages that the master can send to replica</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">commandFlowControl.new-permits-threshold</td>
      <td style="text-align:left">When a replica reaches this threshold in remaining command messages, it
        sends a request with this additional number of command messages to receive.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">queryFlowControl.initial-nr-of-permits</td>
      <td style="text-align:left">Number of query messages that the master can initially send to a replica</td>
      <td
      style="text-align:left">10000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">queryFlowControl.nr-of-new-permits</td>
      <td style="text-align:left">Additional number of query messages that the master can send to replica</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">queryFlowControl.new-permits-threshold</td>
      <td style="text-align:left">When a replica reaches this threshold in remaining command messages, it
        sends a request with this additional number of query messages to receive.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Recovery</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">recoveryfile</td>
      <td style="text-align:left">Start up with a recovery file to update node names in the controldb</td>
      <td
      style="text-align:left">recovery.json</td>
    </tr>
  </tbody>
</table>

