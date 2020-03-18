# External Command Handlers

Command handling functions are most often directly placed on the Aggregate \(as described in more detail [here](modeling/aggregate.md)\). There are situations however where it is not possible nor desired to route a command directly to an Aggregate instance. Message handling functions, like Command Handlers, can however be placed on any object. It is thus possible to instantiate a 'Command Handling Object'.

A Command Handling Object is a simple \(regular\) object, which has `@CommandHandler` annotated methods. Unlike with Aggregates, there is only a _single_ instance of a Command Handling Object, which handles **all** commands of the types it declares in its methods:

```java
import org.axonframework.commandhandling.CommandHandler;
import org.axonframework.modelling.command.Repository;

public class GiftCardCommandHandler {

    // 1.
    private final Repository<GiftCard> giftCardRepository;

    @CommandHandler
    public void handle(RedeemCardCommand cmd) {
        giftCardRepository.load(cmd.getCardId()) // 2.
                          .execute(giftCard -> giftCard.handle(cmd)); // 3.
    }

    // omitted constructor
}
```

In the above snippet we have decided that the `RedeemCardCommand` should no longer be directly handled on the `GiftCard`. Instead, we load the `GiftCard` manually and execute the desired method on it:

1. The `Repository` for the `GiftCard` Aggregate, used for retrieval and storage of an Aggregate. 

   If `@CommandHandler` methods are placed directly on the Aggregate, Axon will automatically know to call the `Repository` to load a given instance. 

   It is thus _not_ mandatory to directly access the `Repository`, but a [design choice](../../architecture-overview/#separation-of-business-logic-and-infrastructure).

2. To load the intended `GiftCard` Aggregate instance, the `Repository#load(String)` method is used. 

   The provided parameter should be the Aggregate identifier.

3. After that Aggregate has been loaded, the `Aggregate#execute(Consumer)` function should be invoked to perform an operation on the Aggregate.

   Using the `execute` function ensure that the Aggregate life cycle is correctly started. 

