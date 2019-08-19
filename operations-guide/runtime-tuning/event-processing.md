# Tuning Event Processing

Typically, application components contain one or more Event Processors,
 which are responsible for processing incoming events. 
The Tracking Processors have configuration aspects that can be changed at runtime,
 to accommodate for changes in the system topology.

## Increasing and decreasing segment counts

Tracking Processors that handle events in multiple threads use Segments to separate the events in the stream
 across these threads in a reliable way. 
However, especially when these threads are spread across multiple instances of a component,
 and the number of instances changes, it may be useful to scale the number of Segments accordingly.
 
To this end, Axon Framework provides a [split and merge API](../../configuring-infrastructure-components/event-processing/event-processors.md#splitting-and-merging-tracking-tokens).
This API can be utilized directly or through Axon Server, where the latter takes required coordination into account.

### Segment tuning through Axon Server

AxonServer provides an API to trigger the increase and decrease of the number of Segments, at runtime,
 across different instances of a processor.
Follow these steps to increase or decrease the number of Segments for a Processor:

* In the UI, navigate to the component overview, and select one of the application instances that contains the component to change the number of segments for.
* In the Component Details screen, scroll down to the list of processors.
* In the list of processors, click the "Scale" icon to open the Scaling dialogue
* In the Scaling dialogue, either click "Increase" or "Decrease" to adjust the number of segments for the processor.

> **Using the REST API**
>
> The exposed buttons in the UI can also be targeted directly through REST calls through curl for example.
> It is recommended to check the Swagger UI for the split and merge endpoint information
>  if such an approach is necessary. 

### Segment tuning through Axon Framework

The Tracking Event Processors in Axon Framework provide methods to increase or decrease the number of segments for that
 particular instance. 
When using this API, one must provide the ID of the segment to increase/decrease. 
Additionally, the instance on which the method is invoked, must be actively processing that segment.

First, the instance of the Tracking Processor must be obtained. 
This can be done using Axon's Configuration API like so:

```java
// The `Configuration` was returned through the `Configurer` or is available as a bean in the Spring Application Context
public TrackingEventProcessor retrieveTrackingProcessor(org.axonframework.config.Configuration axonConfig,
                                                        String processorName) {
    return axonConfig.eventProcessingConfiguration()
                     .eventProcessor(processorName) // This call returns an Optional
                     .filter(eventProcessor -> eventProcessor instanceof TrackingEventProcessor)
                     .map(eventProcessor -> (TrackingEventProcessor) eventProcessor)
                     .orElseThrow(() -> new IllegalStateException(
                             "No Tracking Event Processor found with name " + processorName
                     ));
}
```

Using the above snippet, a split or merge can be called as follows:

```java
int segmentId;
TrackingEventProcessor trackingProcessor = retrieveTrackingProcessor(axonConfig, processorName);

// Split...
CompletableFuture<Boolean> futureResult = trackingProcessor.splitSegment(segmentId);

// Merge...
CompletableFuture<Boolean> futureResult = trackingProcessor.mergeSegment(segmentId);
```

> **Multi instance set up**
>
> If you have several instances of a given Axon application, that regularly means you have duplicated your Tracking Event Processors.
> Such a set up is a regular scenario to require segment tuning.
>
> Note though that especially in such a set up you would need to delegate said split or merge to the correct instance.
> With "correct instance" is meant the instance owning the segment you want to split and merge.
