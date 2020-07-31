# Axon Server SE

This section is split into 3 sub-sections.

* [Using the ready to use Axon Server SE Docker Image](axon-server-se.md#docker-image)
* [Deployment using Docker Compose](axon-server-se.md#docker-compose)
* [Deployment using Kubernetes](axon-server-se.md#kubernetes)

## Docker Image

Axon provides a ready to use [Axon Server SE image](https://hub.docker.com/r/axoniq/axonserver/). The image is built using a compact image from Google’s “distroless” base images at the gcr.io repository, in this case “gcr.io/distroless/java:11”.

To run the provided image the command below can be executed. This starts Axon Server SE in a docker container with exposes the HTTP \(8024\) and GRPC \(8124\) ports to the host.

```bash
$ docker run -d --name  -p 8024:8024 -p 8124:8124 axoniq/axonserver
```

A quick verification of the running docker container can be done by querying the REST API that is available to retrieve configuration information for a running Axon Server instance.

```bash
$ curl -s http://localhost:8024/v1/public/me
```

This displays the information below:

```bash
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
  "name": "{axon-server-name}",
  "hostName": "{axon-server-hostname}"
}
```

The application is installed in the root with a minimal properties file which is depicted below. The “/data” and “/eventdata” directories are created as volumes, and their data will be accessible on the local filesystem somewhere in Docker’s temporary storage tree.

```text
axoniq.axonserver.snapshot.storage=/eventdata
axoniq.axonserver.controldb-path=/data
axoniq.axonserver.pid-file-location=/data
logging.file=/data/axonserver.log
logging.file.max-history=10
logging.file.max-size=10MB
```

### Customization

The directory locations for the volumes can be specified as per your requirements. The image also has a third directory "/config" which is not marked as a volume. This gives you the capability to have an "axonserver.properties" file which can be placed in this location to override the above mentioned settings as well as add new properties similar to a local install.

Assuming that you have a directory "_axonserverse**"**_ which will be the designated location for your volumes and configuration information.

We will first create the sub-directories for the volumes/configuration. We will also add a couple of custom properties \(name/hostname\) to the axonserver.properties file which will be placed in the config sub-directory. As stated above, you can add additional properties to control the configuration.

```text
$ mkdir -p axonserverse/data axonserverse/events axonserverse/config
$ (
> echo axoniq.axonserver.name=axonserver
> echo axoniq.axonserver.hostname=localhost
> ) > axonserverse/config/axonserver.proprties
```

To start the container with the customizations done above, the following command can be executed:

```text
docker run -d --rm --name axonserver -p 8024:8024 -p 8124:8124 -v `pwd`/axonserverse/data:/data -v `pwd`/axonserverse/events:/eventdata -v `pwd`/axonserverse/config:/config axoniq/axonserver
```

Now if you query the API \(utilizing the “curl” command depicted above\) it will show that it is running with name “axonserver” and hostname “localhost”. Also the data directory will contain the ControlDB file, PID file, and a copy of the log output. The “events” directory will have the event and snapshot data.

This completes a basic setup of the Axon Server SE Docker image with implementation of customizations.

## Docker Compose

Running Axon Server SE in docker-compose helps address more complex requirements around distributed scenarios. The following file will help start Axon Server SE with “./data”, “./events”, and “./config” mounted as volumes and the config directory is actually Read-Only.

This again assumes that you have a directory "_axonserverse**"**_ which will be the designated location for your volumes and configuration information.

```text
version: '3.3'
services:
  axonserver:
    image: axoniq/axonserver
    hostname: axonserver
    volumes:
      - axonserver-data:/data
      - axonserver-events:/eventdata
      - axonserver-config:/config:ro
    ports:
      - '8024:8024'
      - '8124:8124'
      - '8224:8224'
    networks:
      - axon-demo

volumes:
  axonserver-data:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/axonserverse/data
  axonserver-events:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/axonserverse/events
  axonserver-config:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/axonserverse/config

networks:
  axon-demo:
```

Starting the Axon Server SE using the docker-compose command is depicted below.

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

An Axon Server SE instance has a clear and persistent identity, in that it saves identifying information about itself in the controlDB. Also, if it is used as an event store, the context’s events will be stored on disk as well, essentially _**Axon Server SE is a stateful application**_.

In the context of Kubernetes that means we want to bind every Axon Server deployment to its own storage volumes, and also to a predictable network identity. Kubernetes provides us with a StatefulSet deployment class which does just that i.e. _StatefulSets_ represent a set of Pods with unique, persistent identities and stable hostnames that is maintained regardless of where they are scheduled.

A sample YAML descriptor is depicted below which defines a StatefulSet for Axon Server and two Services \(axon-server-gui / axonserver\) to provide access to the HTTP and gRPC ports.

```text
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: axonserver
  labels:
    app: axonserver
spec:
  serviceName: axonserver
  replicas: 1
  selector:
    matchLabels:
      app: axonserver
  template:
    metadata:
      labels:
        app: axonserver
    spec:
      containers:
      - name: axonserver
        image: axoniq/axonserver
        imagePullPolicy: Always
        ports:
        - name: grpc
          containerPort: 8124
          protocol: TCP
        - name: gui
          containerPort: 8024
          protocol: TCP
        volumeMounts:
        - name: data
          mountPath: /data
        - name: events
          mountPath: /eventdata
        readinessProbe:
          httpGet:
            path: /actuator/info
            port: 8024
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 1
          failureThreshold: 30
        livenessProbe:
          httpGet:
            path: /actuator/info
            port: 8024
          initialDelaySeconds: 5
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
  volumeClaimTemplates:
    - metadata:
        name: events
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 5Gi
    - metadata:
        name: data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: axonserver-gui
  labels:
    app: axonserver
spec:
  ports:
  - name: gui
    port: 8024
    targetPort: 8024
  selector:
    app: axonserver
  type: LoadBalancer
---
apiVersion: v1
kind: Service
metadata:
  name: axonserver
  labels:
    app: axonserver
spec:
  ports:
  - name: grpc
    port: 8124
    targetPort: 8124
  clusterIP: None
  selector:
    app: axonserver
---
```

Important to note here is that this is a pretty basic descriptor in the sense that it does not have any settings for the amount of memory and/or cpu to reserve for Axon Server SE which you may want to do for long-running deployments.

To deploy, you would need a Kubernetes cluster and access to the [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) utility to help control these clusters. For a development Kubernetes cluster, it is recommended to use [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) or [Red Hat CodeReady Containers](https://github.com/code-ready/crc) which installs a Red Hat OpenShift Kubernetes cluster on your laptop. For production it is recommended to use a managed service like AWS [EKS](https://aws.amazon.com/eks/) / Google's [GKE](https://cloud.google.com/kubernetes-engine) or Azure's [AKS](https://azure.microsoft.com/en-us/services/kubernetes-service).

The first step would be to create a separate namespace for Axon Server SE.

```text
$ kubectl create ns ${axonserverse-ns}
namespace/running-axon-server created
```

The next step would be to deploy to the cluster.

```text
$ kubectl apply -f axonserver.yml -n ${axonserverse-ns}
statefulset.apps/axonserver created
service/axonserver-gui created
service/axonserver created
```

This completes a basic setup to help install Axon Server SE on Kubernetes.
