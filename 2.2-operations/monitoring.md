# 2.2.2 Monitoring

For monitoring, AxonServer includes Spring Actuator endpoints. The documentation can be accessed via
HTTP on the `/actuator` endpoint of AxonServer. 

In particular, you will find that the `/actuator/health` endpoint can be used to verify that the host is up and reports
disk space left. 

Another important endpoint for monitoring is `/actuator/metrics/axon.events.count`. This reports the current number of 
events stored in a node. Under normal operations, this call should give the same result for all nodes in a cluster, 
within a very small margin.

Other, AxonServer-specific endpoints can be browsed through `/swagger-ui.html`. 
