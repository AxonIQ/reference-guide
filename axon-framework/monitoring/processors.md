# Event Processor monitoring

Event processors should be kept an eye on when determining the health and status of your application. 

## Event Tracker Status <a id="event-tracker-status"></a>

Since [Tracking Tokens](events/event-processors/streaming.md#token-store) "track" the progress of a given Streaming Event Processor, they provide a sensible monitoring hook in any Axon application.
Such a hook proves its usefulness when we want to rebuild our view model and we want to check when the processor has caught up with all the events.

To that end the `StreamingEventProcessor` exposes the `processingStatus()` method.
It returns a map where the key is the segment identifier and the value is an "Event Tracker Status".

The Event Tracker Status exposes a couple of metrics:

* The `Segment` it reflects the status of.
* A boolean through `isCaughtUp()` specifying whether it is caught up with the Event Stream.
* A boolean through `isReplaying()` specifying whether the given Segment is [replaying](events/event-processors/streaming.md#replaying-events).
* A boolean through `isMerging()` specifying whether the given Segment is [merging](events/event-processors/streaming.md#splitting-and-merging-segments).
* The `TrackingToken` of the given Segment.
* A boolean through `isErrorState()` specifying whether the Segment is in an error state.
* An optional `Throwable` if the Event Tracker reached an error state.
* An optional `Long` through `getCurrentPosition` defining the current position of the `TrackingToken`.
* An optional `Long` through `getResetPosition` defining the position at reset of the `TrackingToken`.
  This field will be `null` in case the `isReplaying()` returns `false`.
  It is possible to derive an estimated duration of replaying by comparing the current position with this field.
* An optional `Long` through `mergeCompletedPosition()` defining the position on the `TrackingToken` when merging will be completed.
  This field will be `null` in case the `isMerging()` returns `false`.
  It is possible to derive an estimated duration of merging by comparing the current position with this field.

## Metrics <a id="event-processor-metric"></a>

The messages that are processed by an event processor can be monitored with many metrics through one of the [metric modules](metrics.md). 
Each metric type is created for each event processor, creating insights in performance, latency and throughput. 
