# DDD & CQRS Concepts

Axon is heavily based on the principles of Domain-Driven Design \(DDD\) and Command Query Responsibility Segregation. While a full explanation of these concepts is beyond the scope and intent of this reference guide, we do want to provide a summary of the most important concepts in the context of an Axon application.

## Strategic concepts

Strategic concepts are relatively high-level concepts that influence a system's design on a architectural level. It provides the concepts to design boundaries of components and the interaction between them. While they don't directly impact the design of an individual Axon application, the boundaries of such applications are often influenced by these concepts.

### Domains and Subdomains

The formal definition of a Domain in the context of DDD is:

> A sphere of knowledge, influence, or activity. The subject area to which the user applies a program is the domain of the software.

This definition may appear vague, but it does capture the essence very well. The domain is basically the environment in which you build your software. This environment consists of laws, best practices, expectations, traditions, etc, which define what is important and what is not.

Domains can be very big and different areas within a domain might have different influences. For example, in the domain of banking, there is a clear difference between consumer banking, corporate banking and wealth management.

There are numerous techniques that can help you to discover a domain. [Event Storming](https://www.eventstorming.com/book/) is a particularly interesting one. It is a workshop format for quickly exploring complex business domains.

### Model

A model is:

> A system of abstractions that describes selected aspects of a domain and can be used to solve problems related to that domain;

In other words, a model captures what's important and helpful to us in solving a specific problem within our domain. This definition in itself suggests that an application should consist of more than one model, as each problem has a different ideal model to address it.

See [Tactical Concepts](ddd-cqrs-concepts.md#tactical-concepts) for more information about some of the building blocks of a Model in Axon based applications.

### Bounded Context

A context is:

> The setting in which a word or statement appears that determines its meaning.

In other words, the same domain concept may have a different meaning to different people.

Consider, for example, the concept of a Flight. For a passenger, a Flight is the period between departure of an aircraft until the arrival at your destination. The Ground Crew, however, cares about the arrival of a Flight at the Gate, the number of meals, pillows, etc to get on board, and they're done when the flight leaves the gate. For them, departure time is a deadline, not the starting point.

Therefore, there are a number of rules for Models and Contexts:

* Explicitly define the context within which a model applies.
* Explicitly set boundaries in terms of team organization, usage within specific parts of the application, and physical manifestations such as code bases and database schemas.
* Keep the model strictly consistent within these bounds, but don’t be distracted or confused by issues outside it.

### Context Mapping

A bounded context never lives entirely on its own. Information from different contexts will eventually be synchronized. It is useful to model this interaction explicitly. Domain-Driven Design names a few relationships between contexts, which drive the way they interact:

* partnership \(two contexts/teams combine efforts to build interaction\)
* customer-supplier \(two teams in upstream/downstream relationship - upstream can succeed independently of downstream team\)
* conformist \(two teams in upstream/downstream relationship - upstream has no motivation to provide to downstream, and downstream team does not put effort in translation\)
* shared kernel \(explicitly, sharing a part of the model\)
* separate ways \(cut them loose\)
* anti-corruption layer \(the downstream team builds a layer to prevent upstream design to 'leak' into their own models, by transforming interactions\)

In Axon based application, the context defines the boundary in which Events carry value. Some events may be valuable only in the context in which they are published, while others may be valuable even outside. The broader the scope in which an event \(or any message, in that respect\) is published, the more components end up coupling to the sender.

## Tactical concepts

To build a model, DDD \(and CQRS to some extent as well\) provide a number of useful building blocks. Below are a few building blocks that are important in the context of Axon based applications.

### Aggregates

An Aggregate is an entity or group of entities that is always kept in a consistent state \(within a single ACID transaction\). The Aggregate Root is the entity within the aggregate that is responsible for maintaining this consistent state. This makes the aggregate the prime building block for implementing a command model in any CQRS based application.

The formal definition, by DDD is:

> A cluster of associated objects that are treated as a unit for the purpose of data changes. External references are restricted to one member of the Aggregate, designated as the root. A set of consistency rules applies within the Aggregate's boundaries.

In CQRS based applications, Aggregates are very explicitly present in the Command Model, as that is where change is initiated. However, Query Models / Projections are also built up of Aggregates. Generally, however, aggregates in Query Models are much more straightforward, as state invariants are generally less strict in those models.

### Saga

Not every command is able to completely execute in a single atomic transaction. A very common example that pops up quite often as an argument for transactions is the money transfer. It is often believed that an atomic and consistent transaction is absolutely required to transfer money from one account to another. Well, it is not. On the contrary, it is quite impossible to do. What if money is transferred from an account on bank A \(instance A of aggregate `BankAccount`\), to another account on bank B \(instance B of aggregate `BankAccount`\)? Does bank A acquire a lock in bank B's database? If the transfer is in progress, is it strange that bank A has deducted the amount, but bank B has not deposited it yet? Not really, it's "underway". On the other hand, if something goes wrong while depositing the money on bank B's account, bank A's customer would want his money back. So we do expect some form of consistency, eventually. Another example could be `GiftCardPaymentSaga` which would start once the order is placed \(`OrderPlacedEvent`\). It will make sure that once the gift card is successfully redeemed \(`CardRedeemedEvent`\), an order is confirmed \(`ConfirmGiftCardPaymentCommand`\) on the other side.

While ACID transactions are not necessary or even impossible in some cases, some form of transaction management is still required. Typically, these transactions are referred to as BASE transactions: Basically Available, Soft state, Eventual consistency. Contrary to ACID, BASE transactions cannot be easily rolled back. To roll back, compensating actions need to be taken to revert anything that has occurred as part of the transaction. In the gift card example, a redeem failure of `GiftCard`, will reject the `Order` payment.

In CQRS, Sagas can be used to manage these BASE transactions. They respond to events and may dispatch commands, invoke external applications, etc. In the context of Domain-Driven Design, it is common for Sagas to be used as coordination mechanism between different aggregates \(or aggregate instances\) in order to eventually achieve consistency.

### View Models or Projections

In CQRS, View Models \(also known as Projections or Query Models\) are used to efficiently expose information about the application's state. Unlike Command Models, view models focus on data, rather than behavior. View models are generally modeled to accommodate information needs of a specific audience. These models should clearly express the intended audience of the model, to prevent 'distraction' and scope creep, which ultimately leads to loss of maintainability and even performance.

