# Tactial Patterns and Concepts

Inside Bounded Context live tactical paterns and concepts: entities, aggregates, value objects, sagas and others.

## Application Structure

To better understand this concepts we intorduce 'giftcard' domain where customers can issue, redeem or cancel giftcards. Let's quickly visualize potential code structure:

 - com.example.giftcard
   - api
     - `IssueComand.java`
     - `IssuedEvent.java`
     - `RedeemComand.java`
     - `RedeemedEvent.java`
     - `CancelComand.java`
     - `CanceledEvent.java`
   - command
     - `GiftCard.java`
     - `GiftCardTransaction.java`
   - saga
     - `GiftCardPaymentSaga.java`
   - query
     - `CardSummaryProjection.java`
   -  gui
     - `GiftCardController.java`
 - com.example.order
   - api
     - `OrderPlacedEvent.java`
     - `ConfirmGiftCardPaymentCommand.java`
     - `RejectGiftCardPaymentCommand.java`
   - command
    - `Order.java`
  - ...

## Aggregates

An Aggregate is an entity or group of entities that is always kept in a consistent state (within a single ACID transaction). The Aggregate Root is the object on top of the aggregate tree that is responsible for maintaining this consistent state. This makes the aggregate the prime building block for implementing a command model in any CQRS based application.

> **Note**
>
> The term "Aggregate" refers to the aggregate as defined by Evans in Domain-Driven Design:
>
> “A cluster of associated objects that are treated as a unit for the purpose of data changes. External references are restricted to one member of the Aggregate, designated as the root. A set of consistency rules applies within the Aggregate's boundaries.”

For example, a `GiftCard` aggregate could contain other entities: `List<GiftCardTransaction>`. To keep the entire aggregate in a consistent state, adding a transaction (`GiftCardTransaction`) to a gift card should be done via the `GiftCard` entity. In this case, the `GiftCard` entity is the appointed aggregate root.

In Axon, aggregates are identified by an Aggregate Identifier. This may be any object, but there are a few guidelines for good implementations of identifiers. Identifiers must:

* implement `equals` and `hashCode` to ensure good equality comparison with other instances,
* implement a `toString()` method that provides a consistent result \(equal identifiers should provide an equal toString\(\) result\), and
* preferably be `Serializable`.

The test fixtures \(see [Testing](testing.md)\) will verify these conditions and fail a test when an aggregate uses an incompatible identifier. Identifiers of type `String`, `UUID` and the numeric types are always suitable. Do **not** use primitive types as identifiers, as they do not allow for lazy initialization. Axon may, in some circumstances, falsely assume the default value of a primitive to be the value of the identifier.

> **Note**
>
> It is considered a good practice to use randomly generated identifiers, as opposed to sequenced ones. Using a sequence drastically reduces scalability of your application, since machines need to keep each other up-to-date of the last used sequence numbers. The chance of collisions with a UUID is very slim \(a chance of 10−15, if you generate 8.2 × 10 11 UUIDs\).
>
> Furthermore, be careful when using functional identifiers for aggregates. They have a tendency to change, making it very hard to adapt your application accordingly.

## Sagas

Not every command is able to completely execute in a single ACID transaction. A very common example that pops up quite often as an argument for transactions is the money transfer. It is often believed that an atomic and consistent transaction is absolutely required to transfer money from one account to another. Well, it is not. On the contrary, it is quite impossible to do. What if money is transferred from an account on bank A (instance A of aggregate `BankAccount`), to another account on bank B (instance B of aggregate `BankAccount`)? Does bank A acquire a lock in bank B's database? If the transfer is in progress, is it strange that bank A has deducted the amount, but bank B hasn't deposited it yet? Not really, it's "underway". On the other hand, if something goes wrong while depositing the money on bank B's account, bank A's customer would want his money back. So we do expect some form of consistency, eventually. Another example could be `GiftCardPaymentSaga` which would start once the ordrer is placed (`OrderPlacedEvent`). It will make sure that once the gift card is sucessfully reedemed (`CardRedeemedEvent`) an order is confirmed (`ConfirmGiftCardPaymentCommand`) on the other side.

While ACID transactions are not necessary or even impossible in some cases, some form of transaction management is still required. Typically, these transactions are referred to as BASE transactions: Basic Availability, Soft state, Eventual consistency. Contrary to ACID, BASE transactions cannot be easily rolled back. To roll back, compensating actions need to be taken to revert anything that has occurred as part of the transaction. In the gift card example, a redeem failure of `GiftCard`, will reject the `Order` payment.

In CQRS, Sagas can be used to manage these BASE transactions. They respond on events and may dispatch commands, invoke external applications, etc. In the context of Domain-Driven Design, it is common for Sagas to be used as coordination mechanism between different aggregates (or aggregate instances) in order to eventually achieve consistency.

## Eventsourcing

TODO

## View Models 

TODO