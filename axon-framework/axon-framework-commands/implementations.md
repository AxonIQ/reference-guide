# Implementations

Command dispatching, as exemplified in the [Dispatching Commands](command-dispatchers.md) page, has a number of advantages. First of all, there is a single object that clearly describes the intent of the client. By logging the command, you store both the intent and related data for future reference. Command handling also makes it easy to expose your command processing components to remote clients, via web services for example. Testing also becomes a lot easier. You could define test scripts by just defining the starting situation \(given\), command to execute \(when\) and expected results \(then\) by listing a number of events and commands \(see [Testing](../testing/) for more on this\). The last major advantage is that it is very easy to switch between synchronous and asynchronous as well as local versus distributed command processing.

This does not mean command dispatching using explicit command objects is the only way to do it. The goal of Axon is not to prescribe a specific way of working, but to support you doing it your way, while providing best practices as the default behavior. It is still possible to use a service layer that you can invoke to execute commands. The method will just need to start a unit of work \(see [Unit of Work](../messaging-concepts/unit-of-work.md)\) and perform a commit or rollback on it when the method is finished.

The next sections provide an overview of the tasks related to setting up a command dispatching infrastructure with the Axon Framework.

## The Command Gateway

The Command Gateway is a convenient interface towards the command dispatching mechanism. While you are not required to use a gateway to dispatch commands, it is generally the easiest option to do so.

There are two ways to use a Command Gateway. The first is to use the `CommandGateway` interface and the `DefaultCommandGateway` implementation provided by Axon. The command gateway provides a number of methods that allow you to send a command and wait for a result either synchronously, with a timeout or asynchronously.

The other option is perhaps the most flexible of all. You can turn almost any interface into a command gateway using the `CommandGatewayFactory`. This allows you to define your application's interface using strong typing and declaring your own \(checked\) business exceptions. Axon will automatically generate an implementation for that interface at runtime.

### Configuring the Command Gateway

Both your custom Command Gateway and the one provided by Axon need to at least be configured with a Command Bus. In addition, the Command Gateway can be configured with a `RetryScheduler`, `CommandDispatchInterceptor`s, and `CommandCallback`s.

**RetryScheduler**

The `RetryScheduler` is capable of scheduling retries when command execution has failed. When a command fails due to an exception that is explicitly non-transient, no retries are done at all. Note that the retry scheduler is only invoked when a command fails due to a `RuntimeException`. Checked exceptions are regarded as a "business exception" and will never trigger a retry.

Currently two implementations exist:

1. The `IntervalRetryScheduler` will retry a given command at set intervals until it succeeds,

   or a maximum number of retries has taken place.

2. The `ExponentialBackOffIntervalRetryScheduler` retries failed commands with an exponential back-off interval until

   it succeeds, or a maximum number of retries has taken place.

**CommandDispatchInterceptor**

`CommandDispatchInterceptor`s allow modification of `CommandMessage`s prior to dispatching them to the Command Bus. In contrast to `CommandDispatchInterceptor`s configured on the Command Bus, these interceptors are only invoked when messages are sent through this Gateway. For example, these interceptors could be used to attach metadata to a command or perform validation.

**CommandCallback**

A `CommandCallback` can be provided to the Command Gateway upon a regular `send`, specifying what to do with the command handling result. It works with the `CommandMessage` and `CommandResultMessage` classes, thus allowing for some generic behavior for all Commands sent through this gateway regardless of their type.

### Creating a custom Command Gateway

Axon allows a custom interface to be used as a command gateway. The behavior of each method declared in the interface is based on the parameter types, return type and declared exception. Using this gateway is not only convenient, it makes testing a lot easier by allowing you to mock your interface where needed.

This is how parameters affect the behavior of the command gateway:

* The first parameter is expected to be the actual command object to dispatch.
* Parameters annotated with `@MetaDataValue` will have their value assigned to the metadata field with the identifier passed as annotation parameter
* Parameters of type `MetaData` will be merged with the `MetaData` on the `CommandMessage`.

  Metadata defined by latter parameters will overwrite the metadata of earlier parameters, if their key is equal.

* Parameters of type `CommandCallback` will have their `onResult(CommandMessage<? extends C>, CommandResultMessage<? extends R>)` invoked after the command has been handled.
  Although the `CommandCallback` provides a means to deal with the result of command handling, this is no impact on whether you can define a return type on the custom command gateway.
  In case both a callback and return type are defined, the invocations of the callback will always match with the return value \(or exception\).
  Lastly, know that you may pass in several `CommandCallback` instances, which all will be invoked in order. 

* The last two parameters indicate a timeout and may be of types `long` \(or `int`\) and `TimeUnit`.

  The method will block at most as long as these parameters indicate. How the method reacts to a timeout depends on the exceptions declared on the method \(see below\).

  Note that if other properties of the method prevent blocking altogether, a timeout will never occur.

The declared return value of a method will also affect its behavior:

* A `void` return type will cause the method to return immediately, unless there are other indications on the method that one would want to wait, such as a timeout or declared exceptions.
* Return types of `Future`, `CompletionStage` and `CompletableFuture` will cause the method to return immediately.

  You can access the result of the command handler using the `CompletableFuture` instance returned from the method.

  Exceptions and timeouts declared on the method are ignored.

* Any other return type will cause the method to block until a result is available.

  The result is cast to the return type \(causing a `ClassCastException` if the types do not match\).

Exceptions have the following effect:

* Any declared checked exception will be thrown if the command handler \(or an interceptor\) threw an exception of that type.

  If a checked exception is thrown that has not been declared, it is wrapped in a `CommandExecutionException`, which is a `RuntimeException`.

* When a timeout occurs, the default behavior is to return `null` from the method.

  This can be changed by declaring a `TimeoutException`.

  If this exception is declared, a `TimeoutException` is thrown instead.

* When a thread is interrupted while waiting for a result, the default behavior is to return null.

  In that case, the interrupted flag is set back on the thread.

  By declaring an `InterruptedException` on the method, this behavior is changed to throw that exception instead.

  The interrupt flag is removed when the exception is thrown, consistent with the java specification.

* Other runtime exceptions may be declared on the method, but will not have any effect other than clarification to the API user.

Finally, there is the possibility to use annotations:

* As specified in the parameter section, the `@MetaDataValue` annotation on a parameter will have the value of that parameter added as metadata value.

  The key of the metadata entry is provided as parameter to the annotation.

* Methods annotated with `@Timeout` will block at most the indicated amount of time.

  This annotation is ignored if the method declares timeout parameters.

* Classes annotated with `@Timeout` will cause all methods declared in that class to block at most the indicated amount of time, unless they are annotated with their own `@Timeout` annotation or specify timeout parameters.

```java
public interface MyGateway {

    // fire and forget
    void sendCommand(MyPayloadType command);

    // method that attaches metadata and will wait for a result for 10 seconds
    @Timeout(value = 10, unit = TimeUnit.SECONDS)
    ReturnValue sendCommandAndWaitForAResult(MyPayloadType command,
                                             @MetaDataValue("userId") String userId);

    // alternative that throws exceptions on timeout
    @Timeout(value = 20, unit = TimeUnit.SECONDS)
    ReturnValue sendCommandAndWaitForAResult(MyPayloadType command)
                         throws TimeoutException, InterruptedException;

    // this method will also wait, caller decides how long
    void sendCommandAndWait(MyPayloadType command, long timeout, TimeUnit unit)
                         throws TimeoutException, InterruptedException;
}

// To configure a gateway:
CommandGatewayFactory factory = CommandGatewayFactory.builder()
                                                     .commandBus(commandBus)
                                                     .build();
// note that the commandBus can be obtained from the Configuration
// object returned on `configurer.initialize()`.
MyGateway myGateway = factory.createGateway(MyGateway.class);
```

## The Command Bus

The Command Bus is the mechanism that dispatches commands to their respective command handlers within an Axon application. Suggestions on how to use the `CommandBus` can be found [here](command-dispatchers.md#the-command-bus). Several flavors of the command bus, with differing characteristics, exist within the framework:

### AxonServerCommandBus

Axon provides a command bus out of the box, the `AxonServerCommandBus`. It connects to the [AxonIQ AxonServer Server](../../axon-server-introduction.md) to submit and receive Commands.

`AxonServerCommandBus` is a [distributed command bus](command-dispatchers.md#the-command-bus). It uses a [`SimpleCommandBus`](implementations.md) to handle incoming commands on different JVM's by default.

{% tabs %}
{% tab title="Axon Configuration API" %}
Declare dependencies:

```text
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-server-connector</artifactId>
    <version>${axon.version}</version>
</dependency>
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-configuration</artifactId>
    <version>${axon.version}</version>
</dependency>
```

Configure your application:

```java
// Returns a Configurer instance with default components configured. `AxonServerCommandBus` is configured as Command Bus by default.
Configurer configurer = DefaultConfigurer.defaultConfiguration();
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
By simply declaring dependency to `axon-spring-boot-starter`, Axon will automatically configure the Axon Server Command Bus:

```text
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-spring-boot-starter</artifactId>
    <version>${axon.version}</version>
</dependency>
```

> **Excluding the Axon Server Connector**
>
> If you exclude `axon-server-connector` dependency you will fallback to 'non-axon-server' command bus options, the `SimpleCommandBus` \(see [below](implementations.md)\).
{% endtab %}
{% endtabs %}

### SimpleCommandBus

The `SimpleCommandBus` is, as the name suggests, the simplest implementation. It does straightforward processing of commands in the thread that dispatches them. After a command is processed, the modified aggregate\(s\) are saved and generated events are published in that same thread. In most scenarios, such as web applications, this implementation will suit your needs.

Like most `CommandBus` implementations, the `SimpleCommandBus` allows interceptors to be configured. `CommandDispatchInterceptor`s are invoked when a command is dispatched on the command bus. The `CommandHandlerInterceptor`s are invoked before the actual command handler method is, allowing you to do modify or block the command. See [Command Interceptors](../messaging-concepts/message-intercepting.md#command-interceptors) for more information.

Since all command processing is done in the same thread, this implementation is limited to the JVM's boundaries. The performance of this implementation is good, but not extraordinary. To cross JVM boundaries, or to get the most out of your CPU cycles, check out the other `CommandBus` implementations.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
            .configureCommandBus(c -> SimpleCommandBus.builder().transactionManager(c.getComponent(TransactionManager.class)).messageMonitor(c.messageMonitor(SimpleCommandBus.class, "commandBus")).build());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Bean
public SimpleCommandBus commandBus(TransactionManager txManager, AxonConfiguration axonConfiguration) {
    SimpleCommandBus commandBus =
            SimpleCommandBus.builder()
                            .transactionManager(txManager)
                            .messageMonitor(axonConfiguration.messageMonitor(CommandBus.class, "commandBus"))
                            .build();
    commandBus.registerHandlerInterceptor(
            new CorrelationDataInterceptor<>(axonConfiguration.correlationDataProviders())
    );
    return commandBus;
}
```

> **Excluding the Axon Server Connector**
>
> If you exclude the `axon-server-connector` dependency from the `axon-spring-boot-starter` dependency, the `SimpleCommandBus` will be auto-configured for you.
{% endtab %}
{% endtabs %}

### AsynchronousCommandBus

As the name suggest, the `AsynchronousCommandBus` implementation executes commands asynchronously from the thread that dispatches them. It uses an `Executor` to perform the actual handling logic on a different Thread.

By default, the `AsynchronousCommandBus` uses an unbounded cached thread pool. This means a thread is created when a command is dispatched. Threads that have finished processing a command are reused for new commands. Threads are stopped if they have not processed a command for 60 seconds.

Alternatively, an `Executor` instance may be provided to configure a different threading strategy.

Note that the `AsynchronousCommandBus` should be shut down when stopping the application, to make sure any waiting threads are properly shut down. To shut down, call the `shutdown()` method. This will also shutdown any provided `Executor` instance, if it implements the `ExecutorService` interface.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
            .configureCommandBus(c -> AsynchronousCommandBus.builder().transactionManager(c.getComponent(TransactionManager.class))
            .messageMonitor(c.messageMonitor(AsynchronousCommandBus.class, "commandBus"))
            .build());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Bean
public AsynchronousCommandBus commandBus(TransactionManager txManager, AxonConfiguration axonConfiguration) {
    AsynchronousCommandBus commandBus =
            AsynchronousCommandBus.builder()
                            .transactionManager(txManager)
                            .messageMonitor(axonConfiguration.messageMonitor(AsynchronousCommandBus.class, "commandBus"))
                            .build();
    commandBus.registerHandlerInterceptor(new CorrelationDataInterceptor<>(axonConfiguration.correlationDataProviders()));
    return commandBus;
}
```
{% endtab %}
{% endtabs %}

### DisruptorCommandBus

The `SimpleCommandBus` has reasonable performance characteristics. The fact that the `SimpleCommandBus` needs locking to prevent multiple threads from concurrently accessing the same aggregate causes processing overhead and lock contention.

The `DisruptorCommandBus` takes a different approach to multithreaded processing. Instead of having multiple threads each doing the same process, there are multiple threads, each taking care of a piece of the process. The `DisruptorCommandBus` uses the [Disruptor](http://lmax-exchange.github.io/disruptor/), a small framework for concurrent programming, to achieve much better performance, by just taking a different approach to multi-threading. Instead of doing the processing in the calling thread, the tasks are handed off to two groups of threads, that each take care of a part of the processing. The first group of threads will execute the command handler, changing an aggregate's state. The second group will store and publish the events to the event store.

While the `DisruptorCommandBus` easily outperforms the `SimpleCommandBus` by a factor of 4\(!\), there are a few limitations:

* The `DisruptorCommandBus` only supports event sourced aggregates.

  This Command Bus also acts as a Repository for the aggregates processed by the Disruptor.

  To get a reference to the Repository, use `createRepository(AggregateFactory)`.

* A command can only result in a state change in a single aggregate instance.
* When using a cache, it allows only a single aggregate for a given identifier.

  This means it is not possible to have two aggregates of different types with the same identifier.

* Commands should generally not cause a failure that requires a rollback of the unit of work.

  When a rollback occurs, the `DisruptorCommandBus` cannot guarantee that commands are processed in the order they were dispatched.

  Furthermore, it requires a retry of a number of other commands, causing unnecessary computations.

* When creating a new aggregate instance, commands updating that created instance may not all happen in the exact order as provided.

  Once the aggregate is created, all commands will be executed exactly in the order they were dispatched.

  To ensure the order, use a callback on the creating command to wait for the aggregate being created.

  It shouldn't take more than a few milliseconds.

To construct a `DisruptorCommandBus` instance, you need an `EventStore`. This component is explained in the [Event Bus and Event Store](../events/event-bus-and-event-store.md) section.

Optionally, you can provide a `DisruptorConfiguration` instance, which allows you to tweak the configuration to optimize performance for your specific environment:

* `Buffer size` - the number of slots on the ring buffer to register incoming commands.

  Higher values may increase throughput, but also cause higher latency. Must always be a power of 2. Defaults to 4096.

* `ProducerType` - indicates whether the entries are produced by a single thread, or multiple. Defaults to multiple.
* `WaitStrategy` - the strategy to use when the processor threads \(the three threads taking care of the actual processing\) need to wait for each other.

  The best wait strategy depends on the number of cores available in the machine, and the number of other processes running.

  If low latency is crucial, and the `DisruptorCommandBus` may claim cores for itself, you can use the `BusySpinWaitStrategy`.

  To make the command bus claim less of the CPU and allow other threads to do processing, use the `YieldingWaitStrategy`.

  Finally, you can use the `SleepingWaitStrategy` and `BlockingWaitStrategy` to allow other processes a fair share of CPU.

  The latter is suitable if the Command Bus is not expected to be processing full-time.

  Defaults to the `BlockingWaitStrategy`.

* `Executor` - sets the Executor that provides the Threads for the `DisruptorCommandBus`.

  This executor must be able to provide at least four threads.

  Three of the threads are claimed by the processing components of the `DisruptorCommandBus`.

  Extra threads are used to invoke callbacks and to schedule retries in case an Aggregate's state is detected to be corrupt.

  Defaults to a `CachedThreadPool` that provides threads from a thread group called `"DisruptorCommandBus"`.

* `TransactionManager` - defines the transaction manager that should ensure that the storage and publication of events are executed within a transaction.
* `InvokerInterceptors` - defines the `CommandHandlerInterceptor`s that are to be used in the invocation process.

  This is the process that calls the actual Command Handler method.

* `PublisherInterceptors` - defines the `CommandHandlerInterceptor`s that are to be used in the publication process.

  This is the process that stores and publishes the generated events.

* `RollbackConfiguration` - defines on which Exceptions a Unit of Work should be rolled back.

  Defaults to a configuration that rolls back on unchecked exceptions.

* `RescheduleCommandsOnCorruptState` - indicates whether Commands that have been executed against an Aggregate that has been corrupted \(e.g. because a Unit of Work was rolled back\) should be rescheduled.

  If `false` the callback's `onFailure()` method will be invoked.

  If `true` \(the default\), the command will be rescheduled instead.

* `CoolingDownPeriod` - sets the number of seconds to wait to make sure all commands are processed.

  During the cooling down period, no new commands are accepted, but existing commands are processed, and rescheduled when necessary.

  The cooling down period ensures that threads are available for rescheduling the commands and calling callbacks.

  Defaults to `1000` \(1 second\).

* `Cache` - sets the cache that stores aggregate instances that have been reconstructed from the Event Store.

  The cache is used to store aggregate instances that are not in active use by the disruptor.

* `InvokerThreadCount` - the number of threads to assign to the invocation of command handlers.

  A good starting point is half the number of cores in the machine.

* `PublisherThreadCount` - the number of threads to use to publish events.

  A good starting point is half the number of cores, and could be increased if a lot of time is spent on I/O.

* `SerializerThreadCount` - the number of threads to use to pre-serialize events.

  This defaults to `1`, but is ignored if no serializer is configured.

* `Serializer` - the serializer to perform pre-serialization with.

  When a serializer is configured, the `DisruptorCommandBus` will wrap all generated events in a `SerializationAware` message.

  The serialized form of the payload and metadata is attached before they are published to the Event Store.

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
            .configureCommandBus(c ->
                DisruptorCommandBus.builder()
                    .transactionManager(c.getComponent(TransactionManager.class))
                    .messageMonitor(c.messageMonitor(DisruptorCommandBus.class, "commandBus"))
                    .bufferSize(4096)
                    .build() 
            );
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
@Bean
public DisruptorCommandBus commandBus(TransactionManager txManager, AxonConfiguration axonConfiguration) {
    DisruptorCommandBus commandBus =
            DisruptorCommandBus.builder()
                            .transactionManager(txManager)
                            .messageMonitor(axonConfiguration.messageMonitor(DisruptorCommandBus.class, "commandBus"))
                            .build();
    commandBus.registerHandlerInterceptor(new CorrelationDataInterceptor<>(axonConfiguration.correlationDataProviders()));
    return commandBus;
}
```
{% endtab %}
{% endtabs %}

## Distributing the Command Bus

Sometimes, you want multiple instances of command buses in different JVMs to act as one. Commands dispatched on one JVM's command bus should be seamlessly transported to a command handler in another JVM while sending back any results.

That is where the concept of 'distributing the command bus' comes in. The default implementation of a distributed command bus is the `AxonServerCommandBus`. It connects to the [AxonIQ AxonServer Server ](../../axon-server-introduction.md)to submit and receive Commands. Unlike the other `CommandBus` implementations, the `AxonServerCommandBus` does not invoke any handlers at all. All it does is form a "bridge" between command bus implementations on different JVM's.

By default, [`SimpleCommandBus`](implementations.md) is configured to handle incoming commands on the different JVM's. You can configure `AxonServerCommandBus` to use other command bus implementations for this purposes: [`AsynchronousCommandBus`](implementations.md), [`DisruptorCommandBus`](implementations.md).

### DistributedCommandBus

`DistributedCommandBus` is an alternative approach to distributing command bus \(commands\). Each instance of the `DistributedCommandBus` on each JVM is called a "Segment".

![Structure of the Distributed Command Bus](../../.gitbook/assets/distributed-command-bus.png)

The `DistributedCommandBus` relies on two components: a `CommandBusConnector`, which implements the communication protocol between the JVM's, and the `CommandRouter`, which chooses a destination for each incoming command. This router defines which segment of the `DistributedCommandBus` should be given a \`command, based on a routing key calculated by a routing strategy. Two commands with the same routing key will always be routed to the same segment, as long as there is no change in the number and configuration of the segments. Generally, the identifier of the targeted aggregate is used as a routing key.

Two implementations of the `RoutingStrategy` are provided: the `MetaDataRoutingStrategy`, which uses a metadata property in the command message to find the routing key, and the `AnnotationRoutingStrategy`, which uses the `@TargetAggregateIdentifier` annotation on the Command Messages payload to extract the routing key. Obviously, you can also provide your own implementation.

By default, the `RoutingStrategy` implementations will throw an exception when no key can be resolved from a command message. This behavior can be altered by providing a `UnresolvedRoutingKeyPolicy` in the constructor of the `MetaDataRoutingStrategy` or `AnnotationRoutingStrategy`. There are three possible policies:

* `ERROR` - the default, and will cause an exception to be thrown when a Routing Key is not available
* `RANDOM_KEY` - will return a random value when a \`routing key cannot be resolved from the command message.

  This effectively means that those commands will be routed to a random segment of the command bus.

* `STATIC_KEY` - Will return a static key \(being "unresolved"\) for unresolved routing keys.

  This effectively means that all those commands will be routed to the same segment, as long as the configuration of segments does not change.

You can choose different flavor of this components that are available in one of the extension modules:

* [SpringCloud](../../extensions/spring-cloud.md) or 
* [JGroups](../../extensions/jgroups.md).

Configuring a distributed command bus can \(mostly\) be done without any modifications in configuration files.

First of all, the starters for one of the Axon distributed command bus modules needs to be included \(e.g. [JGroups](../../extensions/jgroups.md) or [Spring Cloud](../../extensions/spring-cloud.md)\).

Once that is present, a single property needs to be added to the application context, to enable the distributed command bus:

```text
axon.distributed.enabled=true
```

There is one setting that is independent of the type of connector used:

```text
axon.distributed.load-factor=100
```

The load factor of a Distributed Command Bus is defaulted to `100`.

> **The Load Factor Explained**
>
> The load factor defines the amount of load the instance would carry compared to others. For example, if you have a two machine set up, both with a load factor of 100, both will carry an equal amount of load. Increasing the load factor to 200 on both would still mean that both machines receive the same amount of load. Concluding, the load factor is intended to serve heterogeneous application landscapes with the means to distribute more load to faster machines than to slower machines.

