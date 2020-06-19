# gRPC Metrics

To enable the gRPC metrics, for the Axon Server, the following properties need to be set in the axonserver.properties \(or .yaml file\).

| Property Name | Default Value | Description |
| :--- | :--- | :--- |
| axoniq.axonserver.metrics.grpc.enabled | false | Enables Axon Server gRPC metrics |
| axoniq.axonserver.metrics.grpc.jaeger-enabled | false | Enables exporter for Jaeger |
| axoniq.axonserver.metrics.grpc.jaeger-endpoint |  | Endpoint to access Jaeger exporter. Will not be considered if jaeger-enabled is set to false. |
| axoniq.axonserver.metrics.grpc.jaeger-service-name |  | Service name to be set to Jaeger exporter. |
| axoniq.axonserver.metrics.grpc.prometheus-enabled | false | Enables exporter for Prometheus. |
| axoniq.axonserver.metrics.grpc.z-paged-enabled | false | Enables ZPages for displaying traces/stats. |
| axoniq.axonserver.metrics.grpc.z-pages-port | 8888 | HTTP port to access ZPages. |

