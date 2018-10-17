# 2.2.1 Getting started

## Prerequisites

A Java 8+ JRE should be installed on the system.

## Running Axon Server locally

The Axon Server ZIP download contains executable JAR files for the server itself and the CLI. Copy `axonserver.jar` to a directory of your choice. Because Axon Server uses sensible defaults, you are now ready to go. Start the Axon Server using the following command:

```bash
$ ./axonserver.jar
```

or when not running bash shell:

```bash
$ java -jar axonserver.jar
```

When you see a log line announcing "Started AxonServer in _some-value_ seconds \(JVM running for _some-other-value_\)", the server is ready for action. To verify that the server is started correctly, open the page [http://localhost:8024](http://localhost:8024).

## Running Axon Server in a Docker container

To run Axon Server in Docker you can use the image provided on Docker Hub:

```bash
$ docker run -d --name my-axon-server -p 8024:8024 -p 8124:8124 axoniq/axonserver
```

_WARNING_ This is not a supported image for production purposes. Please use with caution.

If you want to run the clients in Docker containers as well, and are not using something like Kubernetes, use the "`--hostname`" option of the `docker` command to set a useful name like "`axonserver`", and pass the `AXONSERVER_HOSTNAME` environment variable to adjust the properties accordingly:

```bash
$ docker run -d --name my-axon-server -p 8024:8024 -p 8124:8124 --hostname axonserver -e AXONSERVER_HOSTNAME=axonserver axoniq/axonserver
```

When you start the client containers, you can now use "`--link axonserver`" to provide them with the correct DNS entry. The Axon Server-connector looks at the "`axon.axonserver.servers`" property to determine where Axon Server lives, so don't forget to set it to "`axonserver`" and pass it to your app. For more information on the environment variables you can use to tweak settings, see [Customizing the Docker image of Axon Server](properties.md#customizing-the-docker-image-of-axon-server).

## Running Axon Server in Kubernetes and Minikube

_WARNING_: Although you can get a pretty functional cluster running locally using Minikube, you can run into trouble when you want to let it serve clients outside of the cluster. Minikube can provide access to HTTP servers running in the cluster, for other protocols you have to run a special protocol-agnostic proxy like you can with "`kubectl port-forward` _&lt;pod-name&gt;_ _&lt;port-number&gt;_". For non-development scenarios, we don't recommend using Minikube.

Deployment requires the use of a YAML descriptor, defining a StatefulSet for Axon Server, with two Services to provide access to the HTTP and gRPC ports:

```yaml
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
        readinessProbe:
          httpGet:
            port: 8024
            path: /actuator/health
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 1
---
apiVersion: v1
kind: Service
metadata:
  name: axonserver-gui
  labels:
    app: axonserver-gui
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
```

To run it, use the following commands:

```bash
$ kubectl apply -f kubernetes/axonserver.yaml
statefulset.apps "axonserver" created
service "axonserver-gui" created
service "axonserver" created
$ kubectl port-forward axonserver-0 8124
Forwarding from 127.0.0.1:8124 -> 8124
Forwarding from [::1]:8124 -> 8124
```

You can now run the Giftcard app, which will connect throught the proxied gRPC port. To see the Axon Server Web GUI, use "`minikube service --url axonserver-gui`" to obtain the URL for your browser. Actually, if you leave out the "`--url`", minikube will open the the GUI in your default browser for you.

To clean up the deployment, use:

```bash
$ kubectl delete sts axonserver
statefulset.apps "axonserver" deleted
$ kubectl delete svc axonserver
service "axonserver" deleted
$ kubectl delete svc axonserver-gui
service "axonserver-gui" deleted
```

If you're using a 'real' Kubernetes cluster, you'll naturally not want to use "`localhost`" as hostname for Axon Server, so you need to add three lines to the container spec to specify the "`AXONSERVER_HOSTNAME`" setting:

```yaml
...
        readinessProbe:
          httpGet:
            port: 8024
            path: /actuator/health
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 1
        env:
        - name: AXONSERVER_HOSTNAME
          value: axonserver
---
apiVersion: v1
kind: Service
...
```

Use "`axonserver`" \(as that is the name of the Kubernetes service\) if you're going to deploy the client next to the server in the cluster, which is what you'ld probably want. Running the client outside the cluster, with Axon Server _inside_, entails extra work to enable and secure this, and is definitely beyond the scope of this example.

