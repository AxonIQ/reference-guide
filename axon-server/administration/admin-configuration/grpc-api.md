## Event processor administration

Service
name: [EventProcessorAdminService](https://github.com/AxonIQ/axon-server-api/blob/master/src/main/proto/admin.proto)

| Operation                        | Purpose                                                                                                  | Method                                                                               |
|:---------------------------------|:---------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|
| List all even processor          | Provide a list of all event processors defined by the connected applications.                            | rpc GetAllEventProcessors(google.protobuf.Empty) returns (stream EventProcessor)     |
| List event processo by component | Provide a list of all event processors defined by the specified component.                               | rpc GetEventProcessorsByComponent(Component) returns (stream EventProcessor)         |
| Start event processor            | Start a distributed event processor, propagating the start request to all EP instances connected to AS * | rpc StartEventProcessor(EventProcessorIdentifier) returns (AdminActionResult)        |
| Pause event processor            | Pause a distributed event processor, propagating the pause request to all EP instances connected to AS * | rpc PauseEventProcessor(EventProcessorIdentifier) returns (AdminActionResult)        |
| Split event processor segment    | Split the largest known segment of the distributed event processor into two segments.                    | rpc SplitEventProcessor(EventProcessorIdentifier) returns (AdminActionResult)        |
| Merge event processor segments   | Merge the smallest known two segments of the distributed event processor into one. **                    | rpc MergeEventProcessor(EventProcessorIdentifier) returns (AdminActionResult)        |
| List load balance strategies     | Provide a list of all load balancing strategies.                                                         | GetBalancingStrategies(google.protobuf.Empty) returns (stream LoadBalancingStrategy) |
| Load balance event processor     | Balance the load across several instances of an event processor, accordingly to the selected strategy.   | rpc LoadBalanceProcessor(LoadBalanceRequest) returns (stream google.protobuf.Empty)  |
| Set auto load balance strategy   | Define the load balancing strategy to use for automatic load balancing.                                  | SetAutoLoadBalanceStrategy(LoadBalanceRequest) returns (stream google.protobuf.Empty)|

\* Clients need to be already running and connected to AS before the operation is executed.
** It may not work if the two smallest segments are not claimed by applications connected to AS.