# Query Handling

This chapter we will cover the process of handling and dispatching
 [queries](../../configuring-infrastructure-components/messaging-concepts/messaging-concepts.md#queries) within an
 Axon application in more detail.
Queries resembles the "request for data" by some component, typically answered by a "projection" or "view model".
The latter is what is called the
 [Query Model](../../introduction/architecture-overview/ddd-cqrs-concepts.md#view-models-or-projections); 
 a model tailored towards answering queries, updated by event handlers.
