# System Properties

The system configuration can be maintained/supplied in three ways.

* [Configuration File](configuration.md#configuration-file)
* [Command Line](configuration.md#command-line)
* [Environment Variables](configuration.md#environment-variable-s)

### _Configuration File_

The most commonly and preferred way is to have an _axonserver.properties_ or _\_ \_axonserver.yml _\_\*\*\_file which contains the desired configuration parameters. The location of the file should be the current working directory or alternatively can be placed within a "\_config"\_ subdirectory \(relative to the current working directory\).

An important note - In case both files are detected by Axon Server, it will read from both.

### _Command-Line_

In case the server is being started using “java –jar ...”, we can also supply individual configuration properties with _“-Dproperty=value”_

### _Environment Variable\(s\)_

Configuration values can also be supplied using environment variables. The parameter name should be all in upper case with any kebab-case\(-\) / camelCase and snake\_case\(\_\) substituted with _"\_"\_

### _Recommendations_

There are some recommendations around Axon Server EE/SE configuration,

* Use “./axonserver.properties” for common settings.
* Use “./config/axonserver.properties” for environment/node-specific overrides. 
* Use “-D” or environment variables for one-time settings.

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
      <td style="text-align:left"><em><b>Node setup</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">name</td>
      <td style="text-align:left">Unique node name of the Axon Server node.</td>
      <td style="text-align:left">Hostname of the server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">hostname</td>
      <td style="text-align:left">Hostname of the Axon Server node as communicated to clients.</td>
      <td style="text-align:left">Hostname of the server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">internal-hostname</td>
      <td style="text-align:left">Hostname as communicated to other nodes of the cluster.</td>
      <td style="text-align:left">Hostname of the server</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">domain</td>
      <td style="text-align:left">Domain of this node as communicated to clients. Optional, if set will
        be appended to the hostname in communication with clients.</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">internal-domain</td>
      <td style="text-align:left">Domain as communicated to other nodes of the cluster.</td>
      <td style="text-align:left">&quot;domain&quot; value</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">port</td>
      <td style="text-align:left">gRPC port for the Axon Server node.</td>
      <td style="text-align:left">8124</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">server.port <em>(no prefix)</em>
      </td>
      <td style="text-align:left">HTTP port for the Axon Server console.</td>
      <td style="text-align:left">8024</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">internal-port</td>
      <td style="text-align:left">
        <p>gRPC port for communication between Axon Server nodes within a cluster.</p>
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
      <td style="text-align:left">Development mode which allows deleting all events from event store.<em>(Axon SE only)</em>
      </td>
      <td style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">set-web-socket-allowed-origins</td>
      <td style="text-align:left">Set WebSocket CORS Allowed Origins.</td>
      <td style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>File Locations</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.storage</td>
      <td style="text-align:left">Path where (regular) events are stored as segmented files on disk.</td>
      <td
      style="text-align:left">./data directory</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.storage</td>
      <td style="text-align:left">Path where Snapshot Events are stored as segmented files on disk.</td>
      <td
      style="text-align:left">./data directory</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">controldb-path</td>
      <td style="text-align:left">Path where Axon Server&apos;s control database (axonserver-controldb)
        is created.</td>
      <td style="text-align:left">./data directory</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">controldb-backup-location</td>
      <td style="text-align:left">Location where the control DB backups are created.</td>
      <td style="text-align:left">.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">pid-file-location</td>
      <td style="text-align:left">Location where AxonServer creates its pid file.</td>
      <td style="text-align:left">.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.log-storage-folder</td>
      <td style="text-align:left">Directory where the transaction logs for replication are stored.</td>
      <td
      style="text-align:left">./log directory</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">accesscontrol.token-dir</td>
      <td style="text-align:left">Directory where token is stored on startup.</td>
      <td style="text-align:left">./security directory</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>File Names</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.bloom-index-suffix</td>
      <td style="text-align:left">File suffix for bloom files</td>
      <td style="text-align:left">.bloom</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.events-suffix</td>
      <td style="text-align:left">File suffix for events files.</td>
      <td style="text-align:left">.events</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.index-suffix</td>
      <td style="text-align:left">File suffix for index files</td>
      <td style="text-align:left">.index</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.bloom-index-suffix</td>
      <td style="text-align:left">File suffix for snapshot bloom files</td>
      <td style="text-align:left">.sbloom</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.events-suffix</td>
      <td style="text-align:left">File suffix for snapshot files</td>
      <td style="text-align:left">.snapshots</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.index-suffix</td>
      <td style="text-align:left">File suffix for index files for snapshots</td>
      <td style="text-align:left">.sindex</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.index-suffix</td>
      <td style="text-align:left">File suffix for index files for transaction logs</td>
      <td style="text-align:left">.index</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.log-suffix</td>
      <td style="text-align:left">File suffix for transaction log files</td>
      <td style="text-align:left">.log</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Event Store</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.bloom-index-fpp</td>
      <td style="text-align:left">False-positive percentage allowed for bloom index. Decreasing the value
        increases the size of the bloom indexes.</td>
      <td style="text-align:left">0.03</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.force-interval</td>
      <td style="text-align:left">Interval to force syncing files to disk (ms).</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.primary-cleanup-delay</td>
      <td style="text-align:left">Delay to clear ByfeBuffers from off-heap memory for writable segments.</td>
      <td
      style="text-align:left">15</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.secondary-cleanup-delay</td>
      <td style="text-align:left">Delay to clear ByfeBuffers from off-heap memory for read-only segments.</td>
      <td
      style="text-align:left">15</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.segment-size</td>
      <td style="text-align:left">Size for new storage segments.</td>
      <td style="text-align:left">256Mb</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.sync-interval</td>
      <td style="text-align:left">Interval (ms) to check if there are files that are complete and can be
        closed.</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.validation-segments</td>
      <td style="text-align:left">Number of segments to validate to on startup after unclean shutdown.</td>
      <td
      style="text-align:left">10</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.bloom-index-fpp</td>
      <td style="text-align:left">False-positive percentage allowed for bloom index for snapshots. Decreasing
        the value increases the size of the bloom indexes.</td>
      <td style="text-align:left">0.03</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.force-interval</td>
      <td style="text-align:left">Interval to force syncing files to disk (ms).</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.primary-cleanup-delay</td>
      <td style="text-align:left">Delay to clear ByfeBuffers from off-heap memory for writable segments.</td>
      <td
      style="text-align:left">15</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.secondary-cleanup-delay</td>
      <td style="text-align:left">Delay to clear ByfeBuffers from off-heap memory for read-only segments.</td>
      <td
      style="text-align:left">15</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.segment-size</td>
      <td style="text-align:left">Size for new storage segments.</td>
      <td style="text-align:left">256Mb</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.sync-interval</td>
      <td style="text-align:left">Interval (ms) to check if there are files that are complete and can be
        closed.</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.validation-segments</td>
      <td style="text-align:left">Number of snapshot segments to validate to on startup after unclean shutdown.</td>
      <td
      style="text-align:left">10</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">query.limit</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">200</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">new-permits-timeout</td>
      <td style="text-align:left">Timeout for event trackers while waiting for new permits.</td>
      <td style="text-align:left">120000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">blacklisted-send-after</td>
      <td style="text-align:left">Force sending an event on an event tracker after this number of blacklisted
        events. Ensures that the token in the client application is updated.</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">max-events-per-transaction</td>
      <td style="text-align:left">Maximum number of events that can be sent in a single transaction.</td>
      <td
      style="text-align:left">32767</td>
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
        <p><em>(e.g. logging.level.io.axoniq.axonserver = INFO)</em>
        </p>
      </td>
      <td style="text-align:left">WARN level for all packages</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">logging.file <em>(no prefix)</em>
      </td>
      <td style="text-align:left">File name where log entries should be written to. <em>(e.g. logging.file = messaging.log)</em>
      </td>
      <td style="text-align:left">stdout</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">logging.path <em>(no prefix)</em>
      </td>
      <td style="text-align:left">
        <p>Location where log files should be created.</p>
        <p><em>(e.g. logging.path = /var/log)</em>
        </p>
      </td>
      <td style="text-align:left">stdout</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Cluster setup</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">autocluster.first</td>
      <td style="text-align:left">For auto cluster option, set to the internal host name for the first node
        in the cluster.</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">autocluster.contexts</td>
      <td style="text-align:left">For auto cluster option, defines the list of contexts to connect to or
        create.</td>
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
      <td style="text-align:left">Determines whether the server has ssl enabled on the HTTP port.</td>
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
      <td style="text-align:left">Keystore type. (should be PKCS12)</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>server.ssl.key-store</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Location of the keystore.</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>server.ssl.key-store-password</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Password to access the keystore.</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>server.ssl.key-alias</p>
        <p><em>(No prefix)</em>
        </p>
      </td>
      <td style="text-align:left">Alias to be used to access the keystore.</td>
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
      <td style="text-align:left">Determines whether the server has ssl enabled on the gRPC port.</td>
      <td
      style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.cert-chain-file</td>
      <td style="text-align:left">Location of the public certificate file.</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.private-key-file</td>
      <td style="text-align:left">Location of the private key file.</td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.internal-cert-chain-file</td>
      <td style="text-align:left">
        <p>File containing the full certificate chain to be used in internal communication
          between Axon Server nodes.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">ssl.internal-trust-manager-file</td>
      <td style="text-align:left">
        <p>Trusted certificates for verifying the other AxonServer&apos;s certificate.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
      <td style="text-align:left">None</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Access Control</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">accesscontrol.enabled</td>
      <td style="text-align:left">Indicates that access control is enabled for the server.</td>
      <td style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">accesscontrol.cache-ttl</td>
      <td style="text-align:left">Timeout for authenticated tokens.</td>
      <td style="text-align:left">300000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">accesscontrol.internal-token</td>
      <td style="text-align:left">
        <p>Token used to authenticate Axon Server instances in a cluster.</p>
        <p><em>(Axon EE only)</em>
        </p>
      </td>
      <td style="text-align:left">&lt;em&gt;&lt;/em&gt;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">accesscontrol.token</td>
      <td style="text-align:left">
        <p>Token to be used by client applications connecting to Axon Server.</p>
        <p><em>(Axon SE only)</em>
        </p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">accesscontrol.systemtokenfile</td>
      <td style="text-align:left">File containing a predefined system token.</td>
      <td style="text-align:left"></td>
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
      <td style="text-align:left">Maximum size of a message to be sent to the node.</td>
      <td style="text-align:left">4 MB</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">initial-nr-of-permits</td>
      <td style="text-align:left">Number of messages that the server can initially send to a client.</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">nr-of-new-permits</td>
      <td style="text-align:left">Additional number of messages that the server can send to a client.</td>
      <td
      style="text-align:left">500</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">new-permits-threshold</td>
      <td style="text-align:left">When a client reaches this threshold in remaining messages, it sends a
        request with additional number of messages to receive.</td>
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
      <td style="text-align:left">command-flow-control.initial-nr-of-permits</td>
      <td style="text-align:left">Number of command messages that the master can initially send to a replica.</td>
      <td
      style="text-align:left">10000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">command-flow-control.nr-of-new-permits</td>
      <td style="text-align:left">Additional number of command messages that the master can send to replica.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">command-flow-control.new-permits-threshold</td>
      <td style="text-align:left">When a replica reaches this threshold in remaining command messages, it
        sends a request with this additional number of command messages to receive.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">query-flow-control.initial-nr-of-permits</td>
      <td style="text-align:left">Number of query messages that the master can initially send to a replica.</td>
      <td
      style="text-align:left">10000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">query-flow-control.nr-of-new-permits</td>
      <td style="text-align:left">Additional number of query messages that the master can send to replica.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">query-flow-control.new-permits-threshold</td>
      <td style="text-align:left">When a replica reaches this threshold in remaining query messages, it
        sends a request with this additional number of query messages to receive.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Replication</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.flow-buffer</td>
      <td style="text-align:left">Number of unconfirmed append entry messages that may be sent to peer.</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.force-interval</td>
      <td style="text-align:left">Interval to force writes to disk.</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.force-snapshot-on-join</td>
      <td style="text-align:left">Option to force new members to first receive a snapshot when they join
        a cluster.</td>
      <td style="text-align:left">true</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.heartbeat-timeout</td>
      <td style="text-align:left">Leader sends a heartbeat to followers if it has not sent any other messages
        to a follower for this time (in ms)</td>
      <td style="text-align:left">300</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.initial-election-timeout</td>
      <td style="text-align:left">Extra time (in ms) that follower waits initially before moving to candidate
        state.</td>
      <td style="text-align:left">0</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.log-compaction-enabled</td>
      <td style="text-align:left">Deletes old log files when all the entries in the file are applied for
        more than log-retention-hours hour.</td>
      <td style="text-align:left">true</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.log-retention-hours</td>
      <td style="text-align:left">Time to keep log files after all entries have been applied.</td>
      <td style="text-align:left">1</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.max-election-timeout</td>
      <td style="text-align:left">Maximal time (in ms.) that a follower waits before moving to candidate
        state, if it has not received any messages from a leader. Also, time that
        leader waits before stepping down if it has not heard from the majority
        of its followers.</td>
      <td style="text-align:left">2500</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.max-entries-per-batch</td>
      <td style="text-align:left">Maximum number of append entry messages sent to one peer before moving
        to the next.</td>
      <td style="text-align:left">10</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.max-indexes-in-memory</td>
      <td style="text-align:left">Number of index files for replication segments that Axon Server keeps
        in memory</td>
      <td style="text-align:left">10</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">storage.event.max-indexes-in-memory</td>
      <td style="text-align:left">Number of index files for event segments that Axon Server keeps in memory</td>
      <td
      style="text-align:left">50</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.max-indexes-in-memory</td>
      <td style="text-align:left">Number of index files for snapshot segments that Axon Server keeps in
        memory</td>
      <td style="text-align:left">50</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.max-replication-round</td>
      <td style="text-align:left">The number of attempts the log replication process will do during the
        replication a snapshot until the follower is caught up.</td>
      <td style="text-align:left">10</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.max-snapshot-chunks-per-batch</td>
      <td style="text-align:left">Maximum number of objects that can be sent in a single install snapshot
        message</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.min-active-backups</td>
      <td style="text-align:left">When active backup nodes are added to a context, this indicates on how
        many active backup nodes transactions must be confirmed before the transaction
        is ready for commit.</td>
      <td style="text-align:left">1</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.min-election-timeout</td>
      <td style="text-align:left">Minimal time (in ms.) that a follower waits before moving to candidate
        state, if it has not received any messages from a leader.</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.primary-cleanup-delay</td>
      <td style="text-align:left">Windows only. Delay before forcing the primary segment file from memory
        when no longer in use.</td>
      <td style="text-align:left">5</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.secondary-cleanup-delay</td>
      <td style="text-align:left">Windows only. Delay before forcing the other segment files from memory
        when no longer in use.</td>
      <td style="text-align:left">30</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.segment-size</td>
      <td style="text-align:left">Size of a transaction log file.</td>
      <td style="text-align:left">16MB</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.snapshot-flow-buffer</td>
      <td style="text-align:left">Number of unconfirmed install snapshot messages that may be sent to peer.</td>
      <td
      style="text-align:left">50</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.sync-interval</td>
      <td style="text-align:left">Interval to check for files that can be closed.</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">replication.wait-for-leader-timeout</td>
      <td style="text-align:left">Timeout (in ms.) to wait for leader when requesting access to event store
        while leader change in progress, if not set defaults to maxElectionTimeout.</td>
      <td
      style="text-align:left">-1</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Keep Alive</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">keep-alive-time</td>
      <td style="text-align:left">Interval at which AxonServer will send timeout messages. Set to 0 to disable
        gRPC timeout checks.</td>
      <td style="text-align:left">2500</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">keep-alive-timeout</td>
      <td style="text-align:left">Timeout (in ms.) for keep alive messages on gRPC connections.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">min-keep-alive-time</td>
      <td style="text-align:left">Minimum keep alive interval (in ms.) accepted by this end of the gRPC
        connection.</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">client-heartbeat-timeout</td>
      <td style="text-align:left">Timeout (in ms.) on application level heartbeat between client and Axon
        Server.</td>
      <td style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">client-heartbeat-check-initial-delay</td>
      <td style="text-align:left">Initial time delay (in ms.) before Axon Server checks for heartbeats from
        clients.</td>
      <td style="text-align:left">10000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">client-heartbeat-check-rate</td>
      <td style="text-align:left">How often (in ms.) does Axon Server check for heartbeats from clients.</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">heartbeat.enabled</td>
      <td style="text-align:left">If this is set Axon Server will respond to heartbeats from clients and
        send heartbeat</td>
      <td style="text-align:left">false</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Maintenance Tasks</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.connection-check-delay</td>
      <td style="text-align:left">Delay before the first run of the connection checker (in ms.)</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.connection-check-interval</td>
      <td style="text-align:left">Interval between each run of the connection checker (in ms.)</td>
      <td style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.connection-wait-time</td>
      <td style="text-align:left">Timeout for connection request (in ms.)</td>
      <td style="text-align:left">3000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.metrics-distribute-delay</td>
      <td style="text-align:left">Delay before the first run of the metrics distributor (in ms.)</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.metrics-distribute-interval</td>
      <td style="text-align:left">Interval between each run of the metrics distributor (in ms.)</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.rebalance-delay</td>
      <td style="text-align:left">Delay before the first run of the rebalancer (in seconds)</td>
      <td style="text-align:left">7</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.rebalance-interval</td>
      <td style="text-align:left">Interval between each run of the rebalancer (in seconds)</td>
      <td style="text-align:left">15</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.auto-balancing</td>
      <td style="text-align:left">Automatic rebalancing of client connections enabled.</td>
      <td style="text-align:left">true</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cache-close-rate</td>
      <td style="text-align:left">Interval (in ms.) at which to check for timed out queries and commands.</td>
      <td
      style="text-align:left">5000</td>
    </tr>
    <tr>
      <td style="text-align:left"><em><b>Performance</b></em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.max-bloom-filters-in-memory</td>
      <td style="text-align:left">Maximum number of bloom filters to keep in memory</td>
      <td style="text-align:left">100</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.max-indexes-in-memory</td>
      <td style="text-align:left">Maximum number of indexes to keep open in memory</td>
      <td style="text-align:left">50</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">event.read-buffer-size</td>
      <td style="text-align:left">Size of the buffer when reading from non-memory mapped files. (SE only)</td>
      <td
      style="text-align:left">32KB</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.max-bloom-filters-in-memory</td>
      <td style="text-align:left">Maximum number of bloom filters to keep in memory</td>
      <td style="text-align:left">100</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.max-indexes-in-memory</td>
      <td style="text-align:left">Maximum number of indexes to keep open in memory</td>
      <td style="text-align:left">50</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">snapshot.read-buffer-size</td>
      <td style="text-align:left">Size of the buffer when reading from non-memory mapped files. (SE only)</td>
      <td
      style="text-align:left">32KB</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">executor-thread-count</td>
      <td style="text-align:left">Number of threads for executing incoming gRPC requests</td>
      <td style="text-align:left">16</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">command-thread</td>
      <td style="text-align:left">Threads per client responsible for sending commands to the client.</td>
      <td
      style="text-align:left">1</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">query-thread</td>
      <td style="text-align:left">Threads per client responsible for sending queries to the client.</td>
      <td
      style="text-align:left">1</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster-executor-thread-count</td>
      <td style="text-align:left">Number of threads for executing incoming gRPC requests for internal communication</td>
      <td
      style="text-align:left">16</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.query-threads</td>
      <td style="text-align:left">Threads per connected Axon Server node responsible for forwarding queries
        to that node.</td>
      <td style="text-align:left">1</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">cluster.command-threads</td>
      <td style="text-align:left">Threads per connected Axon Server node responsible for forwarding commands
        to that node.</td>
      <td style="text-align:left">1</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">grpc-buffered-messages</td>
      <td style="text-align:left">The initial flow control setting for gRPC level messages. This is the
        number of messages that may may be en-route before the sender stops emitting
        messages. This setting is per-request and only affects streaming requests
        or responses. Application-level flow control settings and buffer restriction
        settings are still in effect.</td>
      <td style="text-align:left">500</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">default-command-timeout</td>
      <td style="text-align:left">Timeout (in ms.) for commands sent to command handler</td>
      <td style="text-align:left">300000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">default-query-timeout</td>
      <td style="text-align:left">Timeout (in ms.) for queries sent to query handler</td>
      <td style="text-align:left">300000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">query.timeout</td>
      <td style="text-align:left">Timeout for ad-hoc queries</td>
      <td style="text-align:left">300000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">websocket-update.rate</td>
      <td style="text-align:left">Settings to influence how often Axon Server (in ms.) sends updates to
        the dashboard for updated metrics or tracking event processor status.</td>
      <td
      style="text-align:left">1000</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">websocket-update.initial-delay</td>
      <td style="text-align:left">On start, it will wait &quot;initial-delay&quot; (in ms.) before sending
        any updates. After that it will check every &quot;rate&quot; milliseconds.</td>
      <td
      style="text-align:left">10000</td>
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
      <td style="text-align:left">Start up with a recovery file to update node names in the controldb.</td>
      <td
      style="text-align:left">recovery.json</td>
    </tr>
  </tbody>
</table>

