# Axon Server

This section is split into 3 sub-sections.

* [Construction of the Axon Server Docker Image](README.md#docker-image)
* [Deployment using Docker Compose ](README.md#docker-compose)and
* [Deployment using Kubernetes](README.md#kubernetes)

> Unified Axon Server Artifact
>
> In an effort to simplify the deployment process, since 2023.1 version we've updated Axon Server to release a single artifact, replacing the previously separate artifacts for the Standard and Enterprise edition.
> Depending on the presence of a license, Axon Server will now automatically adjust.
>

## Docker Image

AxonIQ provides ready to use [Axon Server images](https://hub.docker.com/r/axoniq/axonserver). There are two types of images available: one with Axon Server running as the user "`root`" and one with Axon Server running as user "`axonserver`". Both images are based on [Eclipse Temurin](https://hub.docker.com/_/eclipse-temurin), which means they include a (limited) shell that allows you to connect "into" the running image and perform some commands. 

* The "`root`" image of version 2023.1 is available as "`axoniq/axonserver:2023.1`" and is based on "`eclipse-temurin:11-focal`". This image is particularly useful for running in Docker Desktop, as it will not have any trouble creating files and directories as user "`root`".
* The "`axonserver`" image of version 2023.1 is available as "`axoniq/axonserver:2023.1-nonroot`" and is based on  the same Eclipse Temurin image. This image is more secure and useful in Kubernetes and OpenShift clusters. You should take care to declare the user- and group-id, both of which are `1001` and are named "`axonserver`". Doing this will ensure that any mounted volumes will be writable by the user running Axon Server.

The images export the following volumes:

* "`/axonserver/config`"

  This is where you can add configuration files, such as an additional `axonserver.properties` and the license file. However, you can also opt to use, e.g., Kubernetes or Docker-compose secrets. Note that Axon Server EE assumes it can write to the directory configured with "`axoniq.axonserver.enterprise.licenseDirectory`", so you don't have to put the license on all nodes.
* "`/axonserver/data`"

  This is where the ControlDB, the PID file, and a copy of the application logs are written to.
* "`/axonserver/events`"

  In this volume the Event Store is created, with a single directory per context.
* "`/axonserver/log`"

  In this volume the Replication Logs are created, with a single directory per Replication Group.
* "`/axonserver/exts`"

  In this volume you can place Extension JAR-files, such as the LDAP and OAuth2 extensions.
* "`/axonserver/plugins`"

In this volume Axon Server will place all uploaded plugins.

> _Using the axoniq/axonserver:latest image_ <a name="docker-latest"></a>
> 
> Up to release 2023.2.0 the default Axon Server image (axoniq/axonserver:latest) was using Axon Server Standard Edition version 4.6.11. There is a difference in the directory structure between version 4.6.11 and the current version. One change is that the working directory has changed from `/` to `/axonserver`. All files and directories created by Axon Server are stored in this directory (or subdirectories). 
> 
> The following locations have new default values:
> 
> | Property                                   | Axon Server SE 4.6.11 | Axon Server 2023.2              | 
> |--------------------------------------------|-----------------------|---------------------------------|
> | axoniq.axonserver.event.storage	           | /eventdata            | /axonserver/events              |
> | axoniq.axonserver.snapshot.storage         | /eventdata            | /axonserver/events              |
> | axoniq.axonserver.controldb-path	          | /data                 | /axonserver/data                |
> | axoniq.axonserver.plugin-package-directory | /data/plugins/bundles | /axonserver/plugins/bundles     |
> | axoniq.axonserver.plugin-cache-directory   | /data/plugins/cache   | /axonserver/plugins/cache       |
> | logging.file                               | /data/axonserver.log  | /axonserver/data/axonserver.log |
> 
> As these locations have changed you need to update the volume mappings that you use in the container. The mapping for /eventdata must be changed to /axonserver/events,
and the mapping for /data must be changed to /axonserver/data.

## Building you own Image

A starter Dockerfile is included below which can be tailored as per your requirements.

The starter file helps create the image in multiple stages,

* The image will be based on a compact image from Google’s “distroless” base images at the gcr.io repository, in this case “gcr.io/distroless/java:11”.
* The first stage creates the directories that will become our volumes. This step cannot be performed in the Distroless image, because that image does not provide a shell.
* The second stage begins by copying the home directory with its volume mount points, carefully keeping ownership set to the new user.
* The last steps copy the executable jar named axonserver.jar and a common set of properties. It marks the volume mounting points and exposed ports and finally specifies the command to start Axon Server EE.

```bash
FROM busybox as source

RUN mkdir -p /axonserver/config /axonserver/data /axonserver/events /axonserver/log /axonserver/exts

FROM gcr.io/distroless/java:11

COPY --from=source /axonserver /axonserver
COPY axonserver.jar axonserver.properties /axonserver/

WORKDIR /axonserver

VOLUME [ "/axonserver/config", "/axonserver/data", "/axonserver/events", "/axonserver/log", "/axonserver/exts", "/axonserver/plugins"  ]
EXPOSE 8024/tcp 8124/tcp 8224/tcp

ENTRYPOINT [ "java", "-jar", "./axonserver.jar" ]
```

If you want to build a  "nonroot" version, you need to adjust this as follows:

```text
FROM busybox as source
RUN addgroup -S -g 1001 axonserver \
    && adduser -S -u 1001 -G axonserver -h /axonserver -D axonserver \
    && mkdir -p /axonserver/config /axonserver/data /axonserver/events /axonserver/log /axonserver/exts \
    && chown -R axonserver:axonserver /axonserver

FROM gcr.io/distroless/java:11

COPY --from=source /etc/passwd /etc/group /etc/
COPY --from=source --chown=axonserver /axonserver /axonserver

COPY --chown=axonserver axonserver.jar axonserver.properties /axonserver/

USER axonserver
WORKDIR /axonserver

VOLUME [ "/axonserver/config", "/axonserver/data", "/axonserver/events", "/axonserver/log", "/axonserver/exts", "/axonserver/plugins" ]
EXPOSE 8024/tcp 8124/tcp 8224/tcp

ENTRYPOINT [ "java", "-jar", "./axonserver.jar" ]
```

As you can see this will start by creating the user "`axonserver`" belonging to a group with the same name. When copying the directory, we now have to ensure that ownership transfers correctly and specify the user to run as, but otherwise it looks pretty similar.

For the common properties \(axonserver.properties\), the minimum set can be added to ensure that the volumes get mounted and logs generated. Again these can be tailored as per the deployment requirements.

```text
axoniq.axonserver.event.storage=./events
axoniq.axonserver.snapshot.storage=./events
axoniq.axonserver.replication.log-storage-folder=./log

axoniq.axonserver.enterprise.licenseDirectory=./config
#axoniq.axonserver.accesscontrol.systemtokenfile=./config/axonserver.token

axoniq.axonserver.controldb-path=./data
axoniq.axonserver.pid-file-location=./data

logging.file=./data/axonserver.log
logging.file.max-history=10
logging.file.max-size=10MB
```

Place the Dockerfile, the Axon Server jar file \(axonserver.jar\), the Axon Server client jar file \(axonserver-cli.jar\) and the axonserver.properties in the current directory. Assuming we are building version 2023.1.1, the image can be constructed using the following command:

```bash
$ docker build --tag my-repository/axonserver:2023.1.1.
```

This completes the construction of the Docker image. The image can pushed to your local repository or you could keep it local if you only want to run it on your development machine. The next step is to run it either using [Docker Compose](axon-server.md#docker-compose) or [Kubernetes](readme.md#kubernetes).

If you want to run the docker image for a standalone instance of Axon Server and have it initialized automatically, you can start it with the "axoniq.axonserver.standalone" property set through the environment, for instance:

```bash
$ docker run -dit -e axoniq.axonserver.standalone=true -p 8024:8024 -p 8124:8124 my-repository/axonserver:2023.1.1
```


## Docker Compose

Axon Server is meant to be run in a distributed manner i.e. as a cluster where there will be multiple instances of Axon Server nodes running all interconnected to each other.

The installation process assumes that Docker Compose will be used to run a 3-node Axon Server cluster i.e. running 3 services of the same container image we built above. Let us designate these services as "_axonserver-1_", "_axonserver-2_" and "_axonserver-3_". We will also give a tag to the image that we constructed above as "_my-repository/axonserver:2023.1_".

Each container instance will use separate volumes for “data”, “events”, and “log”. We will use "secrets" to inject the license file, tokens as well as the cluster/context definitions using the [autocluster](../local-installation/axon-server-ee.md#auto-clustering) mechanism. An environment variable is added to tell Axon Server about the location of the license file.

The complete docker-compose file is depicted below.

```text
version: '3.3'
services:
  axonserver-1:
    image: my-repository/axonserver:2023.1
    hostname: axonserver-1
    volumes:
      - axonserver-data1:/axonserver/data
      - axonserver-events1:/axonserver/events
      - axonserver-log1:/axonserver/log
    secrets:
      - source: axoniq-license
        target: /axonserver/config/axoniq.license
      - source: axonserver-properties
        target: /axonserver/config/axonserver.properties
      - source: axonserver-token
        target: /axonserver/config/axonserver.token
    environment:
      - AXONIQ_LICENSE=/axonserver/config/axoniq.license
    ports:
      - '8024:8024'
      - '8124:8124'
      - '8224:8224'
    networks:
      - axon-demo

  axonserver-2:
    image: my-repository/axonserver:2023.1
    hostname: axonserver-2
    volumes:
      - axonserver-data2:/axonserver/data
      - axonserver-events2:/axonserver/events
      - axonserver-log2:/axonserver/log
    secrets:
      - source: axoniq-license
        target: /axonserver/config/axoniq.license
      - source: axonserver-properties
        target: /axonserver/config/axonserver.properties
      - source: axonserver-token
        target: /axonserver/config/axonserver.token
    environment:
      - AXONIQ_LICENSE=/axonserver/config/axoniq.license
    ports:
      - '8025:8024'
      - '8125:8124'
      - '8225:8224'
    networks:
      - axon-demo

  axonserver-3:
    image: my-repository/axonserver:2023.1
    hostname: axonserver-3
    volumes:
      - axonserver-data3:/axonserver/data
      - axonserver-events3:/axonserver/events
      - axonserver-log3:/axonserver/log
    secrets:
      - source: axoniq-license
        target: /axonserver/config/axoniq.license
      - source: axonserver-properties
        target: /axonserver/config/axonserver.properties
      - source: axonserver-token
        target: /axonserver/config/axonserver.token
    environment:
      - AXONIQ_LICENSE=/axonserver/config/axoniq.license
    ports:
      - '8026:8024'
      - '8126:8124'
      - '8226:8224'
    networks:
      - axon-demo

volumes:
  axonserver-data1:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/data1
      o: bind
  axonserver-events1:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/events1
      o: bind
  axonserver-log1:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/log1
      o: bind
  axonserver-data2:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/data2
      o: bind
  axonserver-events2:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/events2
      o: bind
  axonserver-log2:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/log2
      o: bind
  axonserver-data3:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/data3
      o: bind
  axonserver-events3:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/events3
      o: bind
  axonserver-log3:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/log3
      o: bind

networks:
  axon-demo:

secrets:
  axonserver-properties:
    file: ./axonserver.properties
  axoniq-license:
    file: ./axoniq.license
  axonserver-token:
    file: ./axonserver.token
```

The “axonserver-token” secret is used to allow the CLI to talk with nodes. The access control section details the generation of these tokens. A similar approach can be used to configure more secrets for the certificates, and so enable SSL.

The "axonserver.properties" properties file referred to in the secrets’ definition section is depicted below.

```text
axoniq.axonserver.autocluster.first=axonserver-1
axoniq.axonserver.autocluster.contexts=_admin,default
# We highly recommend protecting your installations at all time, hence why access control is enabled here.
# For quick testing, you can disable this, but be sure to switch it big with high priority.
axoniq.axonserver.accesscontrol.enabled=true
axoniq.axonserver.accesscontrol.internal-token=${generated_token}
axoniq.axonserver.accesscontrol.systemtokenfile=/axonserver/config/axonserver.tok
```

Starting Axon Server using the docker-compose command is depicted below.

```text
$ docker-compose up
```

## Kubernetes

**For example purposes only**

The examples below show only one of the ways you could deploy Axon Server to Kubernetes. As discussed in [this Blog article](https://developer.axoniq.io/w/revisiting-axon-server-in-containers), there are many aspects that you need to carefully plan ahaead for. A more complete set of examples can be found in the "[Running Axon Server](https://github.com/AxonIQ/running-axon-server)" GitHub repository. We especially recommend using [the "Singleton StatefulSet" approach](https://github.com/AxonIQ/running-axon-server/tree/master/3-k8s/4-k8s-ssts-tls). Although the complexity of deploying any application to Kubernetes can be overwhelming, we strongly recommend you to study this subject carefully. The examples we provide are not necessarily the best approach for your particular situation, so be careful about copying them without any further modifications, if only because they generate self-signed certificates that have a one-year validity.

###Creating the Secrets and ConfigMap

An important thing to consider is the use of a "nonroot" image. This is due to the fact that volumes are mounted as owned by the mount location’s owner in Docker, while Kubernetes uses a special security context, defaulting to "`root`". Since a "nonroot" image runs Axon Server under its own user, it has no rights on the mounted volume other than “read”. The context can be specified, but only through the user or group’s ID, and not using their name as we did in the image, because that name does not exist in the k8s management context. So we have to adjust the first stage to specify a specific numeric value _\(here we have given 1001\)_ , and then use that value in the security context of the Stateful set which we shall see below.

We would need to supply a licence/token file \(for client applications\) and cluster/context definitions via an axonserver.properties file. Unlike Docker Compose, Kubernetes mounts Secrets and ConfigMaps as directories rather than files, so we need to split license and configuration to two separate locations. For the license secret we can use a new location “/axonserver/license/axoniq.license” and adjust the environment variable to match. For the system token we’ll use “/axonserver/security/token.txt”, and for the properties file we’ll use a ConfigMap that we mount on top of the “/axonserver/config” directory.

These can be created using "kubectl" directly from their respective file as depicted below. It is recommended to create a dedicated namespace before creating the secrets and the config maps.

```text
$ kubectl create secret generic axonserver-license --from-file=./axoniq.license -n ${axonserver-ns}
secret/axonserver-license created
$ kubectl create secret generic axonserver-token --from-file=./axoniq.token -n ${axonserver-ns}
secret/axonserver-token created
$ kubectl create configmap axonserver-properties --from-file=./axonserver.properties -n ${axonserver-ns}
configmap/axonserver-properties created
$
```

In the descriptor we now have to declare the secret, add a volume for it, and mount the secret on the volume. Then a list of volumes has to be added to link the actual license and properties.

### Deploying Axon Server

The complete spec for the Axon Server Stateful set is given below. This includes the security context, the volume mounts, the readiness and liveness probes and finally the volumes.

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
      securityContext:
        runAsUser: 1001
        fsGroup: 1001
      containers:
      - name: axonserver
        image: axoniq/axonserver:latest-dev-nonroot
        imagePullPolicy: IfNotPresent
        ports:
        - name: grpc
          containerPort: 8124
          protocol: TCP
        - name: gui
          containerPort: 8024
          protocol: TCP
        env:
        - name: AXONIQ_LICENSE
          value: "/axonserver/license/axoniq.license"
        volumeMounts:
        - name: data
          mountPath: /axonserver/data
        - name: events
          mountPath: /axonserver/events
        - name: log
          mountPath: /axonserver/log
        - name: config
          mountPath: /axonserver/config
          readOnly: true
        - name: system-token
          mountPath: /axonserver/security
          readOnly: true
        - name: license
          mountPath: /axonserver/license
          readOnly: true
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
      volumes:
        - name: config
          configMap:
            name: axonserver-properties
        - name: system-token
          secret:
            secretName: axonserver-token
        - name: license
          secret:
            secretName: axonserver-license
  volumeClaimTemplates:
    - metadata:
        name: events
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 5Gi
    - metadata:
        name: log
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
    - metadata:
        name: data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
```

The StatefulSet can be applied using the following command \(assuming that the StatefulSet spec is stored in the file "axonserver-sts.yml"\).

```text
$ kubectl apply -f axonserver-sts.yml -n ${axonserver-ns}
statefulset.apps/axonserver created
```

The next step would be to create the two services required for Axon Server i.e. axonserver-gui on 8024 \(HTTP\) and axonserver on 8124 \(gRPC\).

```text
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
  type: ClusterIP
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
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: axonserver
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/affinity-mode: persistent
spec:
  rules:
  - host: axonserver
    http:
      paths:
      - backend:
          serviceName: axonserver-gui
          servicePort: 8024
---
```

The services use an Ingress to allow incoming traffic and can be deployed with the following command \(assuming that the Service\(s\) spec is stored in the file "axonserver-ing.yml"\).

```text
$ kubectl apply -f axonserver-ing.yml -n ${axonserver-ns}
service/axonserver-gui created
service/axonserver created
ingress.networking.k8s.io/axonserver created
```

The final step is to scale out the cluster. The simplest approach, and most often correct one, is to use a scaling factor other than 1, letting Kubernetes take care of deploying several instances. This means we will get several nodes that Kubernetes can dynamically manage and migrate as needed, while at the same time fixing the name and storage. We will get a number suffixed to the name starting at 0, so a scaling factor of 3 gives us “axonserver-0” through “axonserver-2”.

```text
$ kubectl scale sts axonserver -n ${axonserver-ns} --replicas=3
statefulset.apps/axonserver scaled
```

This completes a basic setup to help install Axon Server on Kubernetes. The customer can choose to tailor the entire setup based on their requirements and usage of Kubernetes.

