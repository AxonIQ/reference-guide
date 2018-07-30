# Quick Start

This section will explain how you can obtain the binaries for Axon to get started and how to build your first Axon application. There are currently two ways of obtaining the binaries: either download the binaries from our website or configure a repository for your build system \(Maven, Gradle, etc\).

## Download Axon

You can download the Axon Framework from our downloads page: [axonframework.org/download](http://www.axonframework.org/download).

This page offers a number of downloads. Typically, you would want to use the latest stable release. However, if you're eager to get started using the latest and greatest features, you could consider using the snapshot releases instead. The downloads page contains a number of assemblies for you to download. Some of them only provide the Axon library itself, while others also provide the libraries that Axon depends on. There is also a "full" zip file, which contains Axon, its dependencies, the sources and the documentation, all in a single download.

If you really want to stay on the bleeding edge of development, you can clone the Git repository: [git://github.com/AxonFramework/AxonFramework.git](git://github.com/AxonFramework/AxonFramework.git), or visit [https://github.com/AxonFramework/AxonFramework](https://github.com/AxonFramework/AxonFramework) to browse the sources online.

## Configure Maven

If you use maven as your build tool, you need to configure the correct dependencies for your project. Add the following code in your dependencies section:

```markup
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-core</artifactId>
    <version>${axon.version}</version>
</dependency>
```

Most of the features provided by the Axon Framework are optional and require additional dependencies. We have chosen not to add these dependencies by default, as they would potentially clutter your project with artifacts you don't need.

## Infrastructure requirements

Axon Framework doesn't impose many requirements on the infrastructure. It has been built and tested against Java 8, making that more or less the only requirement.

Since Axon doesn't create any connections or threads by itself, it is safe to run on an Application Server. Axon abstracts all asynchronous behavior by using `Executor`s, meaning that you can easily pass a container managed Thread Pool, for example. If you don't use a full blown Application Server \(e.g. Tomcat, Jetty or a stand-alone app\), you can use the `Executors` class or the Spring Framework to create and configure Thread Pools.

## Let's build our first Axon application

As our domain we'll take a Giftcard management. See the [wikipedia article](https://en.wikipedia.org/wiki/Gift_card) for a basic definition of gift cards. Essentially, there are just two events in the life cycle of a gift card:                                               
* They get issued: a new gift card gets created with some amount of money stored.
* They get redeemed: all or part of the monetary value stored on the gift card is used to purchase something.

### Commands and Events
As mentioned, we need a command to issue the Giftcard. This command will contain the identifier of the Giftcard and the initial amount. Let's build one:

```java
public class IssueCmd {

    private final String id;
    private final Integer amount;
    
    public IssueCmd(String id, Integer amount) {
        this.id = id;
        this.amount = amount;
    }
    
    public String getId() {
        return id;
    }
    
    public Integer getAmount() {
        return amount;
    }
}
```

We'll name an event that will be triggered after this command is successfuly processed `IssuedEvt`:

```java
public class IssuedEvt {

    private final String id;
    private final Integer amount;
    
    public IssuedEvt(String id, Integer amount) {
        this.id = id;
        this.amount = amount;
    }
    
    public String getId() {
        return id;
    }
    
    public Integer getAmount() {
        return amount;
    }
}
```

Similarly, we'll create `RedeemCmd` (do note that `amount` is amount to be deducted from the Giftcard) and `RedeemEvt`:

```java
public class RedeemCmd {

    @TargetAggregateIdentifier // (1)
    private final String id;
    private final Integer amount;
    
    public RedeemCmd(String id, Integer amount) {
        this.id = id;
        this.amount = amount;
    }
    
    public String getId() {
        return id;
    }
        
    public Integer getAmount() {
        return amount;
    }
}
```

(1) `@TargetAggregateIdentifier` annotation is used by Axon to find the correct Giftcard aggregate instance.

```java
public class RedeemedEvt {
    
    private final String id;
    private final Integer amount;
    
    public RedeemedEvt(String id, Integer amount) {
        this.id = id;
        this.amount = amount;
    }
    
    public String getId() {
        return id;
    }
        
    public Integer getAmount() {
        return amount;
    }
}
```

> **Note** Command/Event definition looks a bit cumbersome, this can be avoided using hierarchy between commands/events, or using libraries like [lombok](https://projectlombok.org/), or using [kotlin](https://kotlinlang.org/) which provides more concise way of defining commands/events.
>
> Defined commands and events do not have `equals/hashCode/toString` methods overriden since our example would be lengthy. However, it is highly recommended to do so due to testing/debugging/auditing reasons.

### Aggregate

Now that we have our commands and events which represent the API of our application, we can create Giftcard aggregate:

```java
public class GiftCard {

    @AggregateIdentifier // (1)
    private String id;
    private int remainingValue;

    public GiftCard() {
        // (2)
    }
	
    @CommandHandler // (3)
    public GiftCard(IssueCmd cmd) {        
        if(cmd.getAmount() <= 0) throw new IllegalArgumentException("amount <= 0");
        AggregateLifecycle.apply(new IssuedEvt(cmd.getId(), cmd.getAmount())); // (4)
    }
	
    @EventSourcingHandler // (5)
    public void on(IssuedEvt evt) {
        id = evt.getId();
        remainingValue = evt.getAmount();
    }

    @CommandHandler
    public void handle(RedeemCmd cmd) {
        if(cmd.getAmount() <= 0) throw new IllegalArgumentException("amount <= 0");
        if(cmd.getAmount() > remainingValue) throw new IllegalStateException("amount > remaining value");
        AggregateLifecycle.apply(new RedeemedEvt(id, cmd.getAmount()));
    }

    @EventSourcingHandler
    public void on(RedeemedEvt evt) {
        remainingValue -= evt.getAmount();
    }
}
```

(1) `@AggregateIdentifier` annotation tells Axon that annotated field will be used as identifier of the Aggregate.

(2) If you are using Axon for Event Sourcing, default constructor is needed so Axon can instantiate the Aggregate and apply all sourced events.

(3) Annotation that is put on methods/constructors that handle commands. When an `IssueCmd` is dispatched, annotated constructor will be invoked.

(4) Invoking `AggregateLifecycle.apply` method will apply method on given aggregate (`@EventSourcingHandler` matching this event will be called on aggregate), and then it will be published to the `EventBus`, so other components can react upon it.

(5) Annotation that is put on methods that handle sourced events.

For more details about `@CommandHandler`s and `@EventSourcingHandler`s please check [command model](/part-ii-domain-logic/command-model.md).

> **Note** All business logic / rules are defined in the `@CommandHandler`s, and all state changes are defined in the `@EventSourcingHandler`s. The reason for this is when we want to get the current state of event-sourced Aggregate, we have to apply all sourced events - we have to invoke `@EventSourcingHandler`s. If the state of our Aggregate is changed outside of `@EventSourcingHandler`s it will not be reflected when we do a replay.

### Query Model

Once we have our Aggregate defined which processes commands and fires events, we can create a Query Model (Projection) based on those fired events. Let's say that we want to build a view that has a summary of Giftcards. In order to do that, the view will issue a query to our Query Model to retrieve necessary information about Giftcards. We will use good old `List` Java structure as our storage. The `CardSummary` class could look like this:

```java
public class CardSummary {

    private final String id;
    private final Integer initialAmount;
    private final Integer remainingAmount;

    public CardSummary(String id, Integer initialAmount, Integer remainingAmount) {
        this.id = id;
        this.initialAmount = initialAmount;
        this.remainingAmount = remainingAmount;
    }

    public String getId() {
        return id;
    }

    public Integer getInitialAmount() {
        return initialAmount;
    }

    public Integer getRemainingAmount() {
        return remainingAmount;
    }

    public CardSummary deductAmount(Integer toBeDeducted) {
        return new CardSummary(id, initialAmount, remainingAmount - toBeDeducted);
    }

    @Override
    public String toString() {
        return "CardSummary{" +
                "id='" + id + '\'' +
                ", initialAmount=" + initialAmount +
                ", remainingAmount=" + remainingAmount +
                '}';
    }
```

Before building a Query Model, we should define a query which would retrieve card summaries with predefined size and offset:

```java
public class FetchCardSummariesQuery {

    private final Integer size;
    private final Integer offset;

    public FetchCardSummariesQuery(Integer size, Integer offset) {
        this.size = size;
        this.offset = offset;
    }

    public Integer getSize() {
        return size;
    }

    public Integer getOffset() {
        return offset;
    }
}
```

Query is, as commands and events, a POJO. We can start building our Query Model now.

```java
public class CardSummaryProjection {

    private final List<CardSummary> cardSummaries = new CopyOnWriteArrayList<>();

    @EventHandler // (1)
    public void on(IssuedEvt evt) {
        CardSummary cardSummary = new CardSummary(evt.getId(), evt.getAmount(), evt.getAmount());
        cardSummaries.add(cardSummary);
    }

    @EventHandler
    public void on(RedeemedEvt evt) {
        cardSummaries.stream()
                     .filter(cs -> evt.getId().equals(cs.getId()))
                     .findFirst()
                     .ifPresent(cardSummary -> {
                         CardSummary updatedCardSummary = cardSummary.deductAmount(evt.getAmount());
                         cardSummaries.remove(cardSummary);
                         cardSummaries.add(updatedCardSummary);
                     });
    }

    @QueryHandler // (2)
    public List<CardSummary> fetch(FetchCardSummariesQuery query) {
        return cardSummaries.stream()
                            .skip(query.getOffset())
                            .limit(query.getSize())
                            .collect(Collectors.toList());
    }
}
```

(1) Annotation that is put on event handlers. Usually used to update a query model. When an `IssuedEvt` is published, this method will be invoked. For more details check [event handling](/part-ii-domain-logic/event-handling.md).

(2) Annotation that is used to mark a method as a query handler. For more details check [query handling](/part-ii-domain-logic/query-handling.md).

### Configuration

Having all these components defined, we can start wiring them in the configuration:

```java
CardSummaryProjection projection = new CardSummaryProjection();
EventHandlingConfiguration eventHandlingConfiguration = new EventHandlingConfiguration();
eventHandlingConfiguration.registerEventHandler(c -> projection);

Configuration configuration = DefaultConfigurer.defaultConfiguration()
                                               .configureAggregate(GiftCard.class) // (1)
                                               .configureEventStore(c -> new EmbeddedEventStore(new InMemoryEventStorageEngine())) //(2)
                                               .registerModule(eventHandlingConfiguration) // (3)
                                               .registerQueryHandler(c -> projection) // (4)
                                               .buildConfiguration(); // (5)
```

(1) Aggregate configuration - it will recognize all command/event handlers and wire them up

(2) For the purpose of this quick start, we'll use in-memory event store to store events

(3) Our projection has event and query handlers, that's why we have it registered within `EventHandlingConfiguration` and as a query handler (4)

(5) At this point we are safe to start our configuration by:

```java
configuration.start();
```

> **Note** If you are using Spring, none of these configuration steps are required if you mark `GiftCard` aggregate with `@Aggregate` annotation and `CardSummaryProjection` as Spring `@Component`.

### Running the application

We can obtain `CommandGateway` and `QueryGateway` from configuration:

```java
CommandGateway commandGateway = configuration.commandGateway();
QueryGateway queryGateway = configuration.queryGateway();
```

Let's issue two Giftcards and redeem some money from them:

```java
commandGateway.sendAndWait(new IssueCmd("gc1", 100));
commandGateway.sendAndWait(new IssueCmd("gc2", 50));
commandGateway.sendAndWait(new RedeemCmd("gc1", 10));
commandGateway.sendAndWait(new RedeemCmd("gc2", 20));
```

To get the current state of projections we can issue the query as shown below and print card summaries.

```java
queryGateway.query(new FetchCardSummariesQuery(2, 0), ResponseTypes.multipleInstancesOf(CardSummary.class))
            .get()
            .forEach(System.out::println);
```

The result should look something like this:

```text
CardSummary{id='gc1', initialAmount=100, remainingAmount=90}
CardSummary{id='gc2', initialAmount=50, remainingAmount=30}
```

Congratulations! You made it! Your first Axon application!

## When you're stuck

While implementing your application, you might run into problems, wonder about why certain things are the way they are, or have some questions that need an answer. The Axon Users mailing list is there to help. Just send an email to [axonframework@googlegroups.com](mailto:axonframework@googlegroups.com). Other users as well as contributors to the Axon Framework are there to help with your issues.

If you find a bug, you can report them at [github.com/AxonFramework/AxonFramework/issues](https://github.com/AxonFramework/AxonFramework/issues). When reporting an issue, please make sure you clearly describe the problem. Explain what you did, what the result was and what you expected to happen instead. If possible, please provide a very simple Unit Test \(JUnit\) that shows the problem. That makes fixing it a lot simpler.


