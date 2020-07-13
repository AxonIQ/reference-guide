# Event Segments

Typically, application components contain one or more Event Processors which are responsible for processing incoming events. Tracking Event Processors have configuration aspects that can be changed at runtime to accommodate for changes in the system topology.‌

## Increasing and decreasing segment counts <a id="increasing-and-decreasing-segment-counts"></a>

Tracking Event Processors that handle events in multiple threads use segments to separate the events in the stream across these threads in a reliable way. However, especially when these threads are spread across multiple instances of a component, and the number of instances changes, it may be useful to scale the number of segments accordingly.‌

To this end, Axon Framework provides a [split and merge API](../../axon-framework/events/event-processors.md#splitting-and-merging-tracking-tokens). This API can be utilized directly or through Axon Server, where the latter takes required coordination into account.‌

### Segment tuning through Axon Server <a id="segment-tuning-through-axon-server"></a>

Axon Server provides an API to trigger the increase and decrease of the number of Segments, at runtime, across different instances of a processor. Follow these steps to increase or decrease the number of segments for a processor:‌

* In the UI, navigate to the component overview, and select one of the application instances that contains the component to change the number of segments for.
* In the Component Details screen, scroll down to the list of processors.
* In the list of processors, click the "Scale" icon to open the Scaling dialogue
* In the Scaling dialogue, either click "Increase" or "Decrease" to adjust the number of segments for the processor.

> **Using the REST API**
>
> The exposed buttons in the UI can also be targeted directly through REST calls through curl for example. It is recommended to check the Swagger UI for the split and merge endpoint information if such an approach is necessary.

