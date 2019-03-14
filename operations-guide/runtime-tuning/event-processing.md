# Tuning Event Processing

Typically, application components contain one or more Event Processors, which are responsible for processing incoming 
events. The Tracking Processors have configuration aspects that can be changed at runtime, to accommodate for changes
in the system topology.

## Increasing and decreasing segment counts
Tracking Processors that handle events in multiple threads use Segments to separate the events in the stream across 
these threads in a reliable way. However, especially when these threads are spread across multiple instances of a 
component, and the number of instances changes, it may be useful to scale the number of Segments accordingly.

### Via Axon Server
AxonServer provides an API to trigger the increase and decrease of the number of Segments, at runtime, across different
instances of a processor.

#### Using the User Interface
Follow these steps to increase of decrease the number of Segments for a Processor:

* In the UI, navigate to the component overview, and select one of the application instances that contains the component
to change the number of segments for.
* In the Component Details screen, scroll down to the list of processors.
* In the list of processors, click the "Scale" icon to open the Scaling dialogue
* In the Scaling dialogue, either click "Increase" or "Decrease" to adjust the number of segments for the processor.

#### Using the REST API


### Directly through AxonFramework
The Tracking Event Processors in Axon Framework provide methods to increase the number of segments for that particular
instance. When using this API, one must provide the ID of the segment to increase. The instance on which the method is
invoked, must be actively processing that segment.

First, the instance of the Tracking Processor must be obtained. This can be done using the `Configuration`:
```java
Configuration c = ... // this was returned via the Configurer API, or is available as a bean in the Spring Application Context
TrackingEventProcessor eventProcessor = c.eventProcessingConfiguration()
                                         .eventProcessor("myProcessorName")
                                         .orElseThrow(() -> );
```
