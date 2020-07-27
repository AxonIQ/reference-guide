# Axon Server EE

This section is split into 3 sub-sections.

* [Construction of the Axon Server EE Docker Image](axon-server-ee.md#construction-of-the-image)
* [Deployment using Docker Compose ](axon-server-ee.md#docker-compose)and
* [Deployment using Kubernetes](axon-server-ee.md#kubernetes)

## Construction of the Image

Axon does not provide a public image for Axon Server EE. A starter Dockerfile is included below which can be tailored as per your requirements. This will work for OpenShift, Kubernetes, as well as Docker and Docker Compose.

The starter file helps create the image in multiple stages,

* The image will be based on a compact image from Google’s “distroless” base images at the gcr.io repository, in this case “gcr.io/distroless/java:11”.
* The first stage creates a user and group named “axonserveree” so that we can run Axon Server EE as a non-root user. It also creates the directories that will become our volumes  \( /axonservee\), and finally sets the ownership.
* The second stage begins by copying the account \(in the form of the “passwd” and “group” files\) and the home directory with its volume mount points, carefully keeping ownership set to the new user.
* The last steps copy the executable jar \(axonserver.jar -&gt; the enterprise version\) and a common set of properties. It marks the volume mounting points and exposed ports and finally specifies the command to start Axon Server EE.

```bash
FROM busybox as source
RUN addgroup -S axonserveree \
    && adduser -S -h /axonserveree -D axonserveree \
    && mkdir -p /axonserveree/config /axonserveree/data \
                /axonserveree/events /axonserveree/log \
    && chown -R axonserveree:axonserveree /axonserveree

FROM gcr.io/distroless/java:11

COPY --from=source /etc/passwd /etc/group /etc/
COPY --from=source --chown=axonserveree /axonserveree /axonserveree

COPY --chown=axonserveree axonserver.jar axonserver-cli.jar axonserver.properties /axonserveree/

USER axonserveree
WORKDIR /axonserveree

VOLUME [ "/axonserveree/config", "/axonserveree/data", "/axonserveree/events", "/axonserveree/log" ]
EXPOSE 8024/tcp 8124/tcp 8224/tcp

ENTRYPOINT [ "java", "-jar", "axonserver.jar" ]
```

For the common properties \(axonserver.properties\), the minimum set can be added to ensure that the volumes get mounted and logs generated. Again these can be tailored as per the deployment requirements.

```text
axoniq.axonserver.event.storage=/axonserveree/events
axoniq.axonserver.snapshot.storage=/axonserveree/events
axoniq.axonserver.replication.log-storage-folder=/axonserveree/log
axoniq.axonserver.controldb-path=/axonserveree/data
axoniq.axonserver.pid-file-location=/axonserveree/data

logging.file=/axonserveree/data/axonserver.log
logging.file.max-history=10
logging.file.max-size=10MB
```

Place the Dockerfile, the Axon Server EE jar file \(axonserver.jar\), the Axon Server EE client jar file \(axonserver-cli.jar\) and the axonserver.properties. The image can be constructed using the following command.

```bash
$ docker build --tag ${TAG} .
```

The ${TAG} could be any tag that you would like to give to the Axon Server EE Docker image. This completes the construction of the Docker image. The image can pushed to your local repository or you could keep it local if you only want to run it on your development machine. The next step is to run it either using [Docker Compose](axon-server-ee.md#docker-compose) or [Kubernetes](axon-server-ee.md#kubernetes).

## Docker Compose

Axon Server EE is meant to be run in a distributed manner i.e. as a cluster where there will be multiple instances of Axon Server EE nodes running all interconnected to each other.

The installation process assumes that Docker Compose will be used to run a 3-node Axon Server EE cluster i.e. running 3 services of the same container image we built above. Let us designate these services as "_axonserver-1_", "_axonserver-2_" and "_axonserver-3_". We will also give a tag to the image that we constructed above as "_axonserver-ee:running_".

Each container instance will use separate volumes for “data”, “events”, and “log”. We will use "secrets" to inject the license file, tokens as well as the cluster/context definitions using the [autocluster](../local-installation/axon-server-ee.md#auto-clustering) mechanism. An environment variable is added to tell Axon Server about the location of the license file.

The complete docker-compose file is depicted below.

```text
version: '3.3'
services:
  axonserver-1:
    image: axonserver-ee:running
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
    image: axonserver-ee:running
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
    image: axonserver-ee:running
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
axoniq.axonserver.accesscontrol.enabled=true
axoniq.axonserver.accesscontrol.internal-token=${generated_token}
axoniq.axonserver.accesscontrol.systemtokenfile=/axonserver/config/axonserver.tok
```

Starting Axon Server EE using the docker-compose command is depicted below.

```text
$ docker-compose up
```

## Kubernetes

The deployment of Axon Server EE on Kubernetes essentially follows the same principle as we have seen for Axon Server SE i.e. using Stateful Sets. However to cater to the distributed deployment topology of Axon Server EE, there will be some changes that would need to be done.

The Dockerfile that we [built](axon-server-ee.md#construction-of-the-image) above would need a change. This is due to the fact that volumes are mounted as owned by the mount location’s owner in Docker, while Kubernetes uses a special security context, defaulting to root. Since our EE image runs Axon Server under its own user \(axonserveree\), it has no rights on the mounted volume other than “read”. The context can be specified, but only through the user or group’s ID, and not using their name as we did in the image, because that name does not exist in the k8s management context. So we have to adjust the first stage to specify a specific numeric value _\(here we have given 1001\)_ , and then use that value in the security context of the Stateful set which we shall see below.

The change is depicted below. As before, create the image using _docker build_ and give it a tag \(e.g. axonserveree-running\)

```text
FROM busybox as source
RUN addgroup -S -g 1001 axonserveree \
    && adduser -S -u 1001 -h /axonserveree -D axonserveree \
    && mkdir -p /axonserveree/config /axonserveree/data \
                /axonserveree/events /axonserveree/log \
    && chown -R axonserveree:axonserveree /axonserveree
```

We would need to supply a licence/token file \(for client applications\) and cluster/context definitions via an axonserver.properties file. Unlike Docker Compose, Kubernetes mounts Secrets and ConfigMaps as directories rather than files, so we need to split license and configuration to two separate locations. For the license secret we can use a new location “/axonserver/license/axoniq.license” and adjust the environment variable to match. For the system token we’ll use “/axonserver/security/token.txt”, and for the properties file we’ll use a ConfigMap that we mount on top of the “/axonserver/config” directory.

These can be created using "kubectl" directly from their respective file as depicted below. It is recommended to create a dedicated namespace before creating the secrets and the config maps.

```text
$ kubectl create secret generic axonserveree-license --from-file=./axoniq.license -n ${axonserveree-ns}
secret/axonserver-license created
$ kubectl create secret generic axonserveree-token --from-file=./axoniq.token -n ${axonserveree-ns}
secret/axonserver-token created
$ kubectl create configmap axonserveree-properties --from-file=./axonserver.properties -n ${axonserveree-ns}
configmap/axonserver-properties created
$
```

In the descriptor we now have to declare the secret, add a volume for it, and mount the secret on the volume. Then a list of volumes has to be added to link the actual license and properties.

The complete spec for the Axon Server EE Stateful set is given below. This includes the security context, the volume mounts, the readiness and liveness probes and finally the volumes.

```text
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: axonserveree
  labels:
    app: axonserveree
spec:
  serviceName: axonserveree
  replicas: 1
  selector:
    matchLabels:
      app: axonserveree
  template:
    metadata:
      labels:
        app: axonserveree
    spec:
      securityContext:
        runAsUser: 1001
        fsGroup: 1001
      containers:
      - name: axonserveree
        image: axonserver-ee:running
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
          value: "/axonserveree/license/axoniq.license"
        volumeMounts:
        - name: data
          mountPath: /axonserveree/data
        - name: events
          mountPath: /axonserveree/events
        - name: log
          mountPath: /axonserveree/log
        - name: config
          mountPath: /axonserveree/config
          readOnly: true
        - name: system-token
          mountPath: /axonserveree/security
          readOnly: true
        - name: license
          mountPath: /axonserveree/license
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
            name: axonserveree-properties
        - name: system-token
          secret:
            secretName: axonserveree-token
        - name: license
          secret:
            secretName: axonserveree-license
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

The StatefulSet can be applied using the following command \(assuming that the StatefulSet spec is stored in the file "axonserveree-sts.yml"\).

```text
$ kubectl apply -f axonserver-sts.yml -n ${axonserveree-ns}
statefulset.apps/axonserveree created
```

The next step would be to create the two services required for Axon Server EE i.e. axonserver-gui on 8024 \(HTTP\) and axonserver on 8124 \(gRPC\).

```text
---
apiVersion: v1
kind: Service
metadata:
  name: axonserveree-gui
  labels:
    app: axonserveree
spec:
  ports:
  - name: gui
    port: 8024
    targetPort: 8024
  selector:
    app: axonserveree
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: axonserveree
  labels:
    app: axonserveree
spec:
  ports:
  - name: grpc
    port: 8124
    targetPort: 8124
  clusterIP: None
  selector:
    app: axonserveree
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: axonserveree
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/affinity-mode: persistent
spec:
  rules:
  - host: axonserveree
    http:
      paths:
      - backend:
          serviceName: axonserveree-gui
          servicePort: 8024
---
```

The services use an Ingress to allow incoming traffic and can be deployed with the following command \(assuming that the Service\(s\) spec is stored in the file "axonserveree-ing.yml"\).

```text
$ kubectl apply -f axonserveree-ing.yml -n ${axonserveree-ns}
service/axonserveree-gui created
service/axonserveree created
ingress.networking.k8s.io/axonserveree created
```

The final step is to scale out the cluster. The simplest approach, and most often correct one, is to use a scaling factor other than 1, letting Kubernetes take care of deploying several instances. This means we will get several nodes that Kubernetes can dynamically manage and migrate as needed, while at the same time fixing the name and storage. We will get a number suffixed to the name starting at 0, so a scaling factor of 3 gives us “axonserver-0” through “axonserver-2”.

```text
$ kubectl scale sts axonserveree -n ${axonserveree-ns} --replicas=3
statefulset.apps/axonserveree scaled
```

This completes a basic setup to help install Axon Server EE on Kubernetes. The customer can choose to tailor the entire setup based on their requirements and usage of Kubernetes.

