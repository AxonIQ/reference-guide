# Sagas

In one of the [previous sections](../../architecture-overview/) we introduced the term Saga and briefly explained the pattern as a good mechanism for managing your BASE transactions.

In the next sections we focus on Saga implementation details, testing, error handling, and deadline handling. Additionally, we explain how to make a good decision on when to use a Saga in the first place.

The Axon Framework provides first-class support for long running business transactions or _**Sagas.**_ This section of the reference guide  intends to cover in detail the capabilities that the Axon Framework provides to help facilitate Saga development

A summary of the various sub-sections is given below.

| Sub-Section | Purpose |
| :--- | :--- |
| [Implementation](implementing-saga.md) | Implementation aspects of Sagas using the Axon Framework |
| [Deadline Handling](deadline-handling.md) | Deadline Handling capabilities within the Axon Framework |
| [Associations](managing-associations.md) | Managing Saga associations using the Axon Framework |
| [Infrastructure](saga-infrastructure.md) | Non-Functional Development concerns for Sagas using the Axon Framework |

