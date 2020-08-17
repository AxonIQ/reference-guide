# Axon Server SE

## Binaries

The [Axon Server SE ZIP download](https://download.axoniq.io/axonserver/AxonServer.zip) contains executable JAR files for the server itself and the CLI. Copy `axonserver.jar/axonserver-cli.jar` to a directory of your choice.

## Running Axon Server SE

From the location where the files have been extracted, please run the following command

```text
$ ./axonserver.jar
     _                     ____
    / \   __  _____  _ __ / ___|  ___ _ ____   _____ _ __
   / _ \  \ \/ / _ \| '_ \\___ \ / _ \ '__\ \ / / _ \ '__|
  / ___ \  >  < (_) | | | |___) |  __/ |   \ V /  __/ |
 /_/   \_\/_/\_\___/|_| |_|____/ \___|_|    \_/ \___|_|
 Standard Edition                        Powered by AxonIQ
```

This will start Axon Server SE using the default ports - 8024 for HTTP / 8124 for gRPC.

The HTTP port is used to serve the Management UI and the REST API provided by Axon Server SE. The gRPC port is used by Axon Framework client applications to connect to Axon Server SE. The management UI can be opened at "[http://localhost:8024](http://localhost:8024)" while the REST API is accessible at "[http://localhost:8024/v1](http://localhost:8024/v1)".

The REST API provides an operation at "/v1/public/me" to get the configuration details for a running instance of Axon Server SE. A representation of the response is given below.

```text
{
  "authentication": false,
  "clustered": false,
  "ssl": false,
  "adminNode": true,
  "developmentMode": false,
  "storageContextNames": [
    "default"
  ],
  "contextNames": [
    "default"
  ],
  "httpPort": 8024,
  "grpcPort": 8124,
  "internalHostName": null,
  "grpcInternalPort": 0,
  "name": ${hostname},
  "hostName": ${hostname}
}
```

To summarize,

* By default, access control and SSL is not enabled. The sections below detail on how to configure this.
* Axon Server SE provides a single context named "default" for event storage and message routing. It is not possible to create any other context.
* The default ports are 8024/8124. These values can be changed via [configuration](../../administration/admin-configuration/).
* The name and hostname default to to the hostname of the system Axon Server SE is running on. These values can be changed via [configuration](../../administration/admin-configuration/) \( “axoniq.axonserver.name” / “axoniq.axonserver.hostname”\).
* Clustering is not available in Axon Server SE.
* The "internalHostName" and "grpcInternalPort" are not applicable to Axon Server SE

This completes a quick setup of the Axon Server SE with all the default values. It is now available as an event store and a message router.

## Access Control

As Axon Server is an event store and may contain sensitive data it is always a good practice to enable access control in production and production-like environments.

The [Access Control](../../security/access-control.md) section details the steps required to setup access control in Axon Server SE.

## SSL

Axon Server SE supports TLS/SSL \(Transport Layer Security/Secure Sockets Layer\) to encrypt all of Axon Server SE's network traffic - From Axon Framework client applications to Axon Server SE.

Axon Server SE has two ports \(HTTP/gRPC\) that need to be enabled for SSL and hence there are two different groups of settings to use, once for each port.

The [SSL](../../security/ssl.md) section details the steps required to setup SSL in Axon Server SE.

## Storage

Axon Server SE will by default look in the current directory for a directory named “data”, and inside it a directory “default”. This is where the events and snapshots for the “default” context will be stored.

The location can be customized using the “axoniq.axonserver.event.storage” and “axoniq.axonserver.snapshot.storage” settings. There is also a small database in the “data” directory, which is referred to as the “ControlDB”, and is used for administrative data. This location you can customize by using the “...controldb-path” setting.

The [configuration](../../administration/admin-configuration/) section details the steps required to setup the storage required for Axon Server SE.

## Development Mode

Axon Server SE can be started in development mode which enables some features for development convenience.

This feature can be enabled by configuring the following property:

```text
axoniq.axonserver.devmode.enabled=true
```

## Resetting Events

Whilst creating new features it can be convenient to restore Axon Server to a clean state with no events stored. This can also be helpful when writing and running integration tests against your system. Please note that data which is not stored in Axon Server \(e.g. tracking tokens\) is not deleted from this feature. These will have to be deleted or reset manually.

Resetting Axon Server can be done via the [CLI](../../administration/admin-configuration/command-line-interface.md) as well as the UI and REST interface.

> Note: This feature is disabled when Axon Server is running in cluster mode

