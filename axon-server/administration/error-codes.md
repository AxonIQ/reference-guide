# Error Codes

This page depicts the various Error codes that Axon Server will return in the case of any problems while Processing Client Requests / Message Handling / Administrative Tasks / Cluster Errors.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Area</th>
      <th style="text-align:left">Error Code</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><em>Client Request Processing Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1000</td>
      <td style="text-align:left">
        <p><b>AUTHENTICATION_TOKEN_MISSING</b>
        </p>
        <p>This indicates that the Axon Server has been configured with authentication
          enabled and will reject any instructions from a client application if the
          token is missing.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1001</td>
      <td style="text-align:left">
        <p><b>AUTHENTICATION_INVALID_TOKEN</b>
        </p>
        <p>This indicates that the Axon Server has been configured with authentication
          enabled and will reject any instructions from a client application if an
          invalid token has been supplied.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1002</td>
      <td style="text-align:left">
        <p><b>UNSUPPORTED_INSTRUCTION</b>
        </p>
        <p>This error is returned when the Axon Server cannot recognize the instruction
          passed to it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1003</td>
      <td style="text-align:left">
        <p><b>INSTRUCTION_EXECUTION_ERROR</b>
        </p>
        <p>This error is returned when the Axon Server throws an error during the
          execution of the instruction passed to it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1004</td>
      <td style="text-align:left">
        <p><b>INSTRUCTION_RESULT_TIMEOUT</b>
        </p>
        <p>This indicates that the Axon Server threw an error during the execution
          of the instruction passed to it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1100</td>
      <td style="text-align:left">
        <p><b>NODE_IS_REPLICA</b>
        </p>
        <p>This error is<b> </b>returned when a request from a node to join a cluster
          is received by a node that is no longer leader.</p>
        <p>Normally the node would forward the request to the leader, but if there
          is a leader change during the processing of the join request this may happen.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1300</td>
      <td style="text-align:left">
        <p><b>NO_SUCH_APPLICATION </b>
        </p>
        <p>This error is returned when any client sends any instruction with a specific
          application name that has not been registered with Axon Server.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1301</td>
      <td style="text-align:left">
        <p><b>NO_SUCH_NODE </b>
        </p>
        <p>This error is returned when any client sends any instruction with a specific
          node name that has not been registered with Axon Server.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1302</td>
      <td style="text-align:left">
        <p><b>CONTEXT_NOT_FOUND </b>
        </p>
        <p>This error is returned when any client sends any instruction with a specific
          context that has not been registered with Axon Server.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1304</td>
      <td style="text-align:left">
        <p><b>CONTEXT_EXISTS </b>
        </p>
        <p>This error is returned when there is an instruction to create a specific
          context within Axon Server and it already exists.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1400</td>
      <td style="text-align:left">
        <p><b>NO_AXONSERVER_FOR_CONTEXT</b>
        </p>
        <p>This error is returned when there is no Axon Server node available for
          the specified context.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-1500</td>
      <td style="text-align:left">
        <p><b>AXONSERVER_NODE_NOT_CONNECTED</b>
        </p>
        <p>This<b> </b>error is returned when there is an instruction to create a
          specific context within Axon Server and it already exists.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Input Errors </em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2000</td>
      <td style="text-align:left">
        <p><b>INVALID_SEQUENCE</b>
        </p>
        <p>This<b> </b>error is returned when there is a gap between the sequence
          number for a particular aggregate instance that the client application
          sends and the current sequence number for that instance in the Axon Server
          Event Store.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2001</td>
      <td style="text-align:left">
        <p><b>PAYLOAD_TOO_LARGE </b>
        </p>
        <p>This error is returned when the payload for the message to be executed
          is too large.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2002</td>
      <td style="text-align:left">
        <p><b>TOO_MANY_EVENTS </b>
        </p>
        <p>This error is<b> </b>returned when an ad-hoc query (e.g. from the Search
          Page) has buffered too many rows to send to the client, this error also
          is returned when a single transaction from a client contains more than
          32767 events (due to a limitation in the event store format, we can only
          store MAX_SHORT events in a single transaction).</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2100</td>
      <td style="text-align:left">
        <p><b>NO_LEADER_AVAILABLE </b>
        </p>
        <p>This error is returned when no leader is available to execute any instruction
          sent to the Axon Server cluster.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2101</td>
      <td style="text-align:left">
        <p><b>NOT_RUNNING_IN_CLUSTER</b>
        </p>
        <p>This error<b> </b>is returned when Axon Server is not running in a cluster.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2102</td>
      <td style="text-align:left">
        <p><b>CONTEXT_UPDATE_IN_PROGRESS</b>
        </p>
        <p>This error is returned when an instruction on a specific context is rejected
          as it might be undergoing an update.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2200</td>
      <td style="text-align:left">
        <p><b>INVALID_TRANSACTION_TOKEN </b>
        </p>
        <p>This error is<b> </b>returned when during replication there is a mismatch
          between the transaction token in the new transaction and the transaction
          token that the event store node expects. This means that there is a likely
          inconsistency in the data between the nodes.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2301</td>
      <td style="text-align:left">
        <p><b>CLUSTER_NOT_ALLOWED</b>
        </p>
        <p>This error is<b> </b>returned when an instruction to create a cluster
          is rejected.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2302</td>
      <td style="text-align:left">
        <p><b>CONTEXT_CREATION_NOT_ALLOWED</b>
        </p>
        <p>This<b> </b>error is<b> </b>returned when an instruction to create a context
          on an Axon Server cluster is rejected due to invalid permissions.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2303</td>
      <td style="text-align:left">
        <p><b>NOT_SUPPORTED_IN_DEVELOPMENT</b>
        </p>
        <p>This<b> </b>error is<b> </b>returned when an instruction valid only for
          non-development environments is attempted on a development environment.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2304</td>
      <td style="text-align:left">
        <p><b>CANNOT_DELETE_INTERNAL_CONTEXT</b>
        </p>
        <p>This error is returned when an operation to delete any internal context
          is attempted on an Axon Server cluster.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2305</td>
      <td style="text-align:left">
        <p><b>MAX_CLUSTER_SIZE_REACHED</b>
        </p>
        <p>This error is returned when the maximum size of the cluster is reached.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2306</td>
      <td style="text-align:left">
        <p><b>ALREADY_MEMBER_OF_CLUSTER</b>
        </p>
        <p>This error is returned when<b> </b>the maximum size of the cluster is
          reached.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2307</td>
      <td style="text-align:left">
        <p><b>NOT_A_MEMBER</b>
        </p>
        <p>This error is returned when<b> </b>the maximum size of the cluster is
          reached.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2308</td>
      <td style="text-align:left">
        <p><b>INVALID_CONTEXT_NAME</b>
        </p>
        <p>This error is returned when<b> </b>an instruction is attemped on an invalid
          context.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2310</td>
      <td style="text-align:left">
        <p><b>CANNOT_REMOVE_LAST_NODE </b>
        </p>
        <p>This error is<b> </b>returned when a user tries to remove the last node
          from a replication group. In this case, the user should delete the replication
          group.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2311</td>
      <td style="text-align:left">
        <p><b>INVALID_PROPERTY_VALUE</b>
        </p>
        <p>This error is returned in multiple conditions e.g.<b> </b>when creating
          a context with properties and one of the properties does not have a valid
          value, also when updating a license and the license file is invalid or
          the environment variable.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2500</td>
      <td style="text-align:left">
        <p><b>SAME_NODE_NAME </b>
        </p>
        <p>This error is<b> </b>returned when a node tries to join the cluster with
          the same internal hostname and port as the leader.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2501</td>
      <td style="text-align:left"><b>UNKNOWN_HOST</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2502</td>
      <td style="text-align:left">
        <p><b>CANNOT_JOIN</b> 
        </p>
        <p>This error is returned when a node tries to join the cluster and there
          is an error.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2510</td>
      <td style="text-align:left">
        <p><b>UNKNOWN_ROLE </b>
        </p>
        <p>This error is<b> </b>returned when an unknown role is assigned to a user
          or application. This can only happen when this is done through the REST
          interface directly or through the CLI.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2511</td>
      <td style="text-align:left">
        <p><b>INVALID_QUERY</b>
        </p>
        <p>This error is returned when<b> </b>the user send a query in the search
          window that is not valid</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Rate Limiting Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-3000</td>
      <td style="text-align:left">
        <p><b>EVENT_RATE_EXCEEDED</b>
        </p>
        <p>This error is returned when the number of Event messages sent to Axon
          Server exceeds the processing rate.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Command Handling Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-4000</td>
      <td style="text-align:left">
        <p><b>NO_HANDLER_FOR_COMMAND </b>
        </p>
        <p>This error is returned when a command message instruction is sent to the
          Axon Server and there is no corresponding handler available for it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-4001</td>
      <td style="text-align:left">
        <p><b>CONNECTION_TO_HANDLER_LOST</b> 
        </p>
        <p>This error is returned when Axon Server loses connection to any of the
          Command Handlers.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-4002</td>
      <td style="text-align:left">
        <p><b>COMMAND_TIMEOUT</b>
        </p>
        <p>This error is returned when a command message instruction is sent to the
          Axon Server and there is an error while processing it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-4003</td>
      <td style="text-align:left">
        <p><b>COMMAND_DISPATCH_ERROR </b>
        </p>
        <p>This error is returned when a command messagae instruction is sent to
          the Axon Server and there is an error while dispatching it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Query Handling Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-5000</td>
      <td style="text-align:left">
        <p><b>NO_HANDLER_FOR_QUERY </b>
        </p>
        <p>This error is returned when a query message instruction is sent to the
          Axon Server and there is no corresponding handler available for it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-5002</td>
      <td style="text-align:left">
        <p><b>QUERY_DISPATCH_ERROR</b>
        </p>
        <p>This error is returned when a query instruction is sent to the Axon Server
          and there is an error while dispatching it.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Backup Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-7000</td>
      <td style="text-align:left">
        <p><b>NODE_NOT_READY_FOR_BACKUP</b>
        </p>
        <p>This<b> </b>error is returned when an Axon Server node is not available
          for any backup operation.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Internal Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9000</td>
      <td style="text-align:left">
        <p><b>DATAFILE_READ_ERROR</b>
        </p>
        <p>This error is returned when Axon Server is unable to read from the Event/Snapshot
          Data Files.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9001</td>
      <td style="text-align:left">
        <p><b>INDEX_READ_ERROR</b>
        </p>
        <p>This error is returned when Axon Server is unable to read from the Index
          files.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9100</td>
      <td style="text-align:left">
        <p><b>DATAFILE_WRITE_ERROR </b>
        </p>
        <p>This error<b> </b>is returned when Axon Server is unable to write to the
          Event/Snapshot Data files.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9101</td>
      <td style="text-align:left">
        <p><b>INDEX_WRITE_ERROR </b>
        </p>
        <p>This error<b> </b>is returned when Axon Server is unable to write to the
          Index files.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9102</td>
      <td style="text-align:left">
        <p><b>DIRECTORY_CREATION_FAILED </b>
        </p>
        <p>This error<b> </b>is returned when Axon Server is unable to create a directory
          for storing events/snapshots or indexes.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9200</td>
      <td style="text-align:left">
        <p><b>VALIDATION_FAILED</b>
        </p>
        <p>This error is returned during startup of Axon Server when it performs
          a validation of the most recent event store segments. This error code is
          returned when the validation fails.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9900</td>
      <td style="text-align:left">
        <p><b>TRANSACTION_ROLLED_BACK</b> 
        </p>
        <p>This error<b> </b>is returned when any transaction is rolled back.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-9500</td>
      <td style="text-align:left">
        <p><b>INTERRUPTED </b>
        </p>
        <p>This error is returned when Axon Server process is stopped while waiting
          for events to be written to the event store segment.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-6000</td>
      <td style="text-align:left">
        <p><b>NO_EVENTSTORE</b>
        </p>
        <p>This error<b> </b>is returned when the Axon Server Event Store is not
          available to perform any instructions.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-6001</td>
      <td style="text-align:left">
        <p><b>CLIENT_DISCONNECTED </b>
        </p>
        <p>This error<b> </b>is returned when an Axon Framework client application
          disconnects from the Axon Server.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Cluster Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-10001</td>
      <td style="text-align:left">
        <p><b>SERVER_TOO_SLOW</b>
        </p>
        <p>This error is returned when any instruction to update the Axon Server
          cluster fails as the server is too slow to respond.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-10002</td>
      <td style="text-align:left">
        <p><b>UNCOMMITTED_CONFIGURATION </b>
        </p>
        <p>This error is returned when an instruction to update configuration for
          the Axon Server cluster fails.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-10007</td>
      <td style="text-align:left">
        <p><b>UNCOMMITTED_TERM </b>
        </p>
        <p>This error is returned when<b> </b>a request to update the configuration
          of a replication group is received before there are any actions committed.
          The request is refused to prevent potential replication issues.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-10003</td>
      <td style="text-align:left">
        <p><b>REPLICATION_TIMEOUT </b>
        </p>
        <p>This error is returned when the replication process between the nodes
          of the Axon Server cluster times out.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><em>Other Errors</em>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-0001</td>
      <td style="text-align:left">
        <p><b>OTHER</b>
        </p>
        <p>Any other<b> </b>errors.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">AXONIQ-2610</td>
      <td style="text-align:left">
        <p><b>SCHEDULED_EVENT_NOT_FOUND</b>
        </p>
        <p>This error is returned when a scheduled event is not found.</p>
      </td>
    </tr>
  </tbody>
</table>

