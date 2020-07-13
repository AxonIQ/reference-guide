# Non-Axon Server to Axon Server

The Axon EE Server package contains a migration tool to migrate from an already existing RDBMS event store to a new Axon EE Server instance. The tool reads events and snapshots from the existing store and pushes them to an Axon EE Server.

The migration tool maintains the state of its migration, so it can be run multiple times.

## Pre-Upgrade Process

* Provision an Axon Server EE cluster with the required number of nodes setup. Startup all the nodes.
* On any one of the admin nodes, create a new file `application.properties`under the following location ${axon\_ee\_server\_home} containing the properties which define the existing event store and the target Axon Server server
  * `axoniq.axonserver.servers` - comma separated list of hostnames and ports for the Axon Server cluster.
  * `axoniq.datasource.eventstore.url` - url of the JDBC data store containing the existing event store
  * `axoniq.datasource.eventstore.username` - username to connect to the JDBC data store containing the existing event store
  * `axoniq.datasource.eventstore.password` - password to connect to the JDBC data store containing the existing event store
  * `axon.serializer.events*=jackson`- The default settings expect the data in the current event store to be serialized using the `XstreamSerializer`. Add this property if the data is serialized using the JacksonSerializer.
* Create a folder called libs under the ${axon\_ee\_server\_home}. Depending upon the type of Database \(Postgres/MySql\), the required JDBC driver jar files should be placed in this directory.

## Upgrade process

* Run the command `axonserver-migration.jar`
* The time to migrate data will vary depending upon the size of the existing data store and the number of nodes setup within the cluster.

## Verification

* Logon to the Axon Server EE console and query the store to check if the migrated event data is present.

## Notes

* The migration tool only migrates the event store data to Axon Server. It does not update the tracking token values in token\_entry tables. Tracking tokens are highly dependent on the implementation of the actual event store used. Migrating them is case specific and error prone. Our recommendation is to reset the tracking processors after the migration.
* The migration tool is like a regular Axon Framework application so properties can be setup accordingly \(e.g. Access Control/Tokens\).

