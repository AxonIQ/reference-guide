# Axon Server SE

This section is split into 3 sub-sections.

* [Deploying the ready to use Axon Server SE Docker Image](axon-server-se.md#docker-images)
* [Deployment using Docker Compose](axon-server-se.md#docker-compose)
* [Deployment using Kubernetes](axon-server-se.md#kubernetes)
* [Deployment using Helm](axon-server-se#using-helm)

## Docker Images

AxonIQ provides a ready to use [Axon Server SE image on Docker Hub](https://hub.docker.com/r/axoniq/axonserver/). There are several variations available:

### **DEPRECATED:** The "distroless" images

The images that use only the version as tag, including "`latest`", have been built using the Google "Distroless Java" images as base. These images are unique in being _very_ compact, in the sense that they only contain the files needed to run a single Java application, but nothing else. The Axon Server SE images use “`gcr.io/distroless/java:11`”, which means they run Axon Server using a Java 11 JVM.

#### Features

The distroless images will only run on recent Intel and AMD processors, using the architecture commonly referred to as "`amd64`".

Axon Server is installed in the root directory, using the following properties file:

```properties
axoniq.axonserver.event.storage=/eventdata
axoniq.axonserver.snapshot.storage=/eventdata
axoniq.axonserver.controldb-path=/data
axoniq.axonserver.pid-file-location=/data
axoniq.axonserver.plugin-package-directory=/data/plugins/bundles
axoniq.axonserver.plugin-cache-directory=/data/plugins/cache
logging.file=/data/axonserver.log
logging.file.max-history=10
logging.file.max-size=10MB
```

The "`/data`" and "`/eventdata`" directories are exported as Docker volumes and will be mapped to directories on the host even if you do not provide alternative locations. To override properties, you can provide a directory with a file named "`axonserver.properties`" in it and mount that as "`/config`" in the container.

### The Eclipse Temurin based images

All other images, recognizable through tag names that consist of multiple, dash-separated elements, have been built using the "`eclipse-temurin`" multi-platform images. These are significantly larger than the Distroless ones, but a safe and widely used alternative. For Axon Server they provide the important benefit of supporting not only the 64-bit Intel architecture, but also 32-bit and 64-bit ARM, which means they can run on a wider range of hardware, including Raspberry Pis and Apple M1 based systems. For development and debugging scenarios the biggest advantage is that they also contain a shell (Bash), which means that you can "connect into" a running container.

**NOTE:** From a security perspective the presence of the shell can be considered a risk. If this is a problem for you, you should build your own image using the examples provided in the "[Running Axon Server](https://github.com/AxonIQ/running-axon-server)" GitHub repository.

#### Features

The Eclipse Temurin based images all run Axon Server in its own home directory, at "`/axonserver`". The properties file contains the following settings:

```properties
axoniq.axonserver.event.storage=/axonserver/events
axoniq.axonserver.snapshot.storage=/axonserver/events
axoniq.axonserver.controldb-path=/axonserver/data
axoniq.axonserver.pid-file-location=/axonserver/data
axoniq.axonserver.plugin-package-directory=/axonserver/plugins/bundles
axoniq.axonserver.plugin-cache-directory=/axonserver/plugins/cache
logging.file=/axonserver/data/axonserver.log
logging.file.max-history=10
logging.file.max-size=10MB
```

The "`/axonserver/config`", "`/axonserver/data`", "`/axonserver/events`", and "`/axonserver/plugins`" directories are exported as Docker volumes and will be mapped to directories on the host even if you do not provide alternative locations. To override properties, you can provide a directory with a file named "`axonserver.properties`" in it and mount that as "`/axonserver/config`" in the container.

#### Variants

The Eclipse Temurin based images are available in several variants, shown by the tag:

* "`<version>-dev`" or "`latest-dev`"

  These images run Axon Server as user "`root`" with a Java 11 JVM. **Note** the "`-dev`" fragment is used in all Eclipse Temurin based images to indicate the presence of a shell.
* Tags with "`-jdk-8`", "`-jdk-11`", or "`jdk-17`"

  These images run Axon Server with the indicated Java version.
* Tags with "`-nonroot`"

  These images run Axon Server as user "`axonserver`", with user-id (UID) `1001` and group-id (GID) `1001`.

##Running Axon Server

To run the provided image the command below can be executed. This starts Axon Server SE in a docker container with exposes the HTTP (by default 8024) and GRPC (by default 8124) ports to the host.

```bash
$ docker run -d --name axonserver -p 8024:8024 -p 8124:8124 axoniq/axonserver
```

A quick verification of the running docker container can be done by querying the REST API that is available to retrieve configuration information for a running Axon Server instance.

```bash
$ curl -s http://localhost:8024/actuator/info
```

If Axon Server is running, this will display the information below:

```json
{"app":{"name":"Axon Server","description":"AxonIQ","version":"4.5.11"}}
```

### Customization

Assuming that you have a directory named  "`axon-server-se`" which will be the designated location for your volumes and configuration information, we will first create the subdirectories for the volumes. We will also add a couple of custom properties (name/hostname) to the axonserver.properties file which will be placed in the config subdirectory. As stated above, you can add additional properties to control the configuration.

```text
$ mkdir -p axon-server-se/data axon-server-se/events axon-server-se/config
$ (
> echo axoniq.axonserver.name=axonserver
> echo axoniq.axonserver.hostname=localhost
> ) > axon-server-se/config/axonserver.properties
```

To start the container with the customizations done above, the following command can be executed:

```text
docker run -d --rm --name axonserver -p 8024:8024 -p 8124:8124 -v `pwd`/axon-server-se/data:/axonserver/data -v `pwd`/axon-server-se/events:/axonserver/events -v `pwd`/axon-server-se/config:/axonserver/config axoniq/axonserver:latest-dev
```

Now if you query the API (utilizing the “`curl`” command depicted above) it will show that it is running with name “`axonserver`” and hostname “`localhost`”. Also, the "`data`" directory will contain the ControlDB file, PID file, and a copy of the log output, while the “`events`” directory will have the event and snapshot data.

This completes a basic setup of the Axon Server SE Docker image with implementation of customizations.

## Docker Compose

Running Axon Server SE in docker-compose helps address more complex requirements around distributed scenarios. The following file will help start Axon Server SE with “`./axon-server-se/data`”, “`./axon-server-se/events`”, and “`./axon-server-se/config`” mounted as volumes. The config directory is mounted Read-Only.

This again assumes that you have a directory "`./axon-server-se`", which will be the location for your volumes and configuration information.

```text
version: '3.3'
services:
  axonserver:
    image: axoniq/axonserver:latest-dev
    hostname: axonserver
    volumes:
      - axonserver-data:/axonserver/data
      - axonserver-events:/axonserver/events
      - axonserver-config:/axonserver/config:ro
    ports:
      - '8024:8024'
      - '8124:8124'
    networks:
      - axon-demo

volumes:
  axonserver-data:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/axon-server-se/data
  axonserver-events:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/axon-server-se/events
  axonserver-config:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/axon-server-se/config

networks:
  axon-demo:
```

Starting Axon Server SE using the `docker-compose` command is shown below.

```text
$ docker-compose up
Creating network "docker-compose_axon-demo" with the default driver
Creating volume "docker-compose_axonserver-data" with local driver
Creating volume "docker-compose_axonserver-events" with local driver
Creating volume "docker-compose_axonserver-config" with local driver
Creating docker-compose_axonserver_1 ... done
Attaching to docker-compose_axonserver_1
axonserver_1  |      _                     ____
axonserver_1  |     / \   __  _____  _ __ / ___|  ___ _ ____   _____ _ __
axonserver_1  |    / _ \  \ \/ / _ \| '_ \\___ \ / _ \ '__\ \ / / _ \ '__|
axonserver_1  |   / ___ \  >  < (_) | | | |___) |  __/ |   \ V /  __/ |
axonserver_1  |  /_/   \_\/_/\_\___/|_| |_|____/ \___|_|    \_/ \___|_|
axonserver_1  |  Standard Edition                        Powered by AxonIQ
```

## Kubernetes

An Axon Server SE instance has a clear and persistent identity, in that it saves identifying information about itself in the controlDB. Also, if it is used as an event store, the context’s events will be stored on disk as well, which means that _Axon Server SE is a stateful application_.

In the context of Kubernetes that means we want to bind every Axon Server deployment to its own storage volumes, and also to a predictable network identity. Kubernetes provides us with a `StatefulSet` deployment class which does just that.

Several sample YAML descriptors are provided in the "[Running Axon Server](https://github.com/AxonIQ/running-axon-server)" GitHub repository.

To deploy, you would need a Kubernetes cluster and access to the [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) utility to help control these clusters. For a development Kubernetes cluster, it is recommended to use [Docker for Desktop](https://docker.com), [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), or [Red Hat CodeReady Containers](https://github.com/code-ready/crc) which installs a Red Hat OpenShift Kubernetes cluster on your laptop. For production, it is recommended to use a managed service like AWS [EKS](https://aws.amazon.com/eks/) / Google's [GKE](https://cloud.google.com/kubernetes-engine) or Azure's [AKS](https://azure.microsoft.com/en-us/services/kubernetes-service).

## Using Helm

Helm (version 3.0 and up) is a very popular tool to install applications in Kubernetes. In the "[Running Axon Server](https://github.com/AxonIQ/running-axon-server)" GitHub repository we provide an example Helm chart that you can customize to your liking.