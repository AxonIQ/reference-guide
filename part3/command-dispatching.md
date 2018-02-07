Command Dispatching
===================

The use of an explicit command dispatching mechanism has a number of advantages. First of all, there is a single object that clearly describes the intent of the client. By logging the command, you store both the intent and related data for future reference. Command handling also makes it easy to expose your command processing components to remote clients, via web services for example. Testing also becomes a lot easier, you could define test scripts by just defining the starting situation (given), command to execute (when) and expected results (then) by listing a number of events and commands (see [Testing](../part2/testing.md)). The last major advantage is that it is very easy to switch between synchronous and asynchronous as well as local versus distributed command processing.

This doesn't mean Command dispatching using explicit command object is the only way to do it. The goal of Axon is not to prescribe a specific way of working, but to support you doing it your way, while providing best practices as the default behavior. It is still possible to use a Service layer that you can invoke to execute commands. The method will just need to start a unit of work (see [Unit of Work](../part1/messaging-concepts.md#unit-of-work)) and perform a commit or rollback on it when the method is finished.

The next sections provide an overview of the tasks related to setting up a Command dispatching infrastructure with the Axon Framework.

The Command Gateway
===================

The Command Gateway is a convenient interface towards the Command dispatching mechanism. While you are not required to use a Gateway to dispatch Commands, it is generally the easiest option to do so.

There are two ways to use a Command Gateway. The first is to use the `CommandGateway` interface and the `DefaultCommandGateway` implementation provided by Axon. The command gateway provides a number of methods that allow you to send a command and wait for a result either synchronously, with a timeout or asynchronously.

The other option is perhaps the most flexible of all. You can turn almost any interface into a Command Gateway using the `CommandGatewayFactory`. This allows you to define your application's interface using strong typing and declaring your own (checked) business exceptions. Axon will automatically generate an implementation for that interface at runtime.

Configuring the Command Gateway
-------------------------------

Both your custom gateway and the one provided by Axon need to be configured with at least access to the Command Bus. In addition, the Command Gateway can be configured with a `RetryScheduler`, `CommandDispatchInterceptor`s, and `CommandCallback`s.

The `RetryScheduler` is capable of scheduling retries when command execution has failed. The `IntervalRetryScheduler` is an implementation that will retry a given command at set intervals until it succeeds, or a maximum number of retries is done. When a command fails due to an exception that is explicitly non-transient, no retries are done at all. Note that the retry scheduler is only invoked when a command fails due to a `RuntimeException`. Checked exceptions are regarded "business exception" and will never trigger a retry. Typical usage of a `RetryScheduler` is when dispatching commands on a Distributed Command Bus. If a node fails, the Retry Scheduler will cause a command to be dispatched to the next node capable of processing the command (see [Distributing the Command Bus](#distributing-the-command-bus)).

`CommandDispatchInterceptor`s allow modification of `CommandMessage`s prior to dispatching them to the Command Bus. In contrast to `CommandDispatchInterceptor`s configured on the CommandBus, these interceptors are only invoked when messages are sent through this gateway. The interceptors can be used to attach meta data to a command or do validation, for example.

The `CommandCallback`s are invoked for each command sent. This allows for some generic behavior for all Commands sent through this gateway, regardless of their type.

Creating a Custom Command Gateway
---------------------------------

Axon allows a custom interface to be used as a Command Gateway. The behavior of each method declared in the interface is based on the parameter types, return type and declared exception. Using this gateway is not only convenient, it makes testing a lot easier by allowing you to mock your interface where needed.

This is how parameters affect the behavior of the CommandGateway:

-   The first parameter is expected to be the actual command object to dispatch.

-   Parameters annotated with `@MetaDataValue` will have their value assigned to the meta data field with the identifier passed as annotation parameter

-   Parameters of type `MetaData` will be merged with the `MetaData` on the CommandMessage. Meta data defined by latter parameters will overwrite the meta data of earlier parameters, if their key is equal.

-   Parameters of type `CommandCallback` will have their `onSuccess` or `onFailure` invoked after the Command is handled. You may pass in more than one callback, and it may be combined with a return value. In that case, the invocations of the callback will always match with the return value (or exception).

-   The last two parameters may be of types `long` (or `int`) and `TimeUnit`. In that case the method will block at most as long as these parameters indicate. How the method reacts on a timeout depends on the exceptions declared on the method (see below). Note that if other properties of the method prevent blocking altogether, a timeout will never occur.

The declared return value of a method will also affect its behavior:

-   A `void` return type will cause the method to return immediately, unless there are other indications on the method that one would want to wait, such as a timeout or declared exceptions.

-   Return types of `Future`, `CompletionStage` and `CompletableFuture` will cause the method to return immediately. You can access the result of the Command Handler using the `CompletableFuture` instance returned from the method. Exceptions and timeouts declared on the method are ignored.

-   Any other return type will cause the method to block until a result is available. The result is cast to the return type (causing a ClassCastException if the types don't match).

Exceptions have the following effect:

-   Any declared checked exception will be thrown if the Command Handler (or an interceptor) threw an exception of that type. If a checked exception is thrown that has not been declared, it is wrapped in a `CommandExecutionException`, which is a `RuntimeException`.

-   When a timeout occurs, the default behavior is to return `null` from the method. This can be changed by declaring a `TimeoutException`. If this exception is declared, a `TimeoutException` is thrown instead.

-   When a Thread is interrupted while waiting for a result, the default behavior is to return null. In that case, the interrupted flag is set back on the Thread. By declaring an `InterruptedException` on the method, this behavior is changed to throw that exception instead. The interrupt flag is removed when the exception is thrown, consistent with the java specification.

-   Other Runtime Exceptions may be declared on the method, but will not have any effect other than clarification to the API user.

Finally, there is the possibility to use annotations:

-   As specified in the parameter section, the `@MetaDataValue` annotation on a parameter will have the value of that parameter added as meta data value. The key of the meta data entry is provided as parameter to the annotation.

-   Methods annotated with `@Timeout` will block at most the indicated amount of time. This annotation is ignored if the method declares timeout parameters.

-   Classes annotated with `@Timeout` will cause all methods declared in that class to block at most the indicated amount of time, unless they are annotated with their own `@Timeout` annotation or specify timeout parameters.

``` java
public interface MyGateway {

    // fire and forget
    void sendCommand(MyPayloadType command);

    // method that attaches meta data and will wait for a result for 10 seconds
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
CommandGatewayFactory factory = new CommandGatewayFactory(commandBus);
// note that the commandBus can be obtained from the `Configuration` object returned on `configurer.initialize()`.
MyGateway myGateway = factory.createGateway(MyGateway.class);
```

The Command Bus
===============

The Command Bus is the mechanism that dispatches commands to their respective Command Handlers. Each Command is always sent to exactly one command handler. If no command handler is available for the dispatched command, a `NoHandlerForCommandException` exception is thrown. Subscribing multiple command handlers to the same command type will result in subscriptions replacing each other. In that case, the last subscription wins.

Dispatching commands
--------------------

The CommandBus provides two methods to dispatch commands to their respective handler: `dispatch(commandMessage, callback)` and `dispatch(commandMessage)`. The first parameter is a message containing the actual command to dispatch. The optional second parameter takes a callback that allows the dispatching component to be notified when command handling is completed. This callback has two methods: `onSuccess()` and `onFailure()`, which are called when command handling returned normally, or when it threw an exception, respectively.

The calling component may not assume that the callback is invoked in the same thread that dispatched the command. If the calling thread depends on the result before continuing, you can use the `FutureCallback`. It is a combination of a `Future` (as defined in the java.concurrent package) and Axon's `CommandCallback`. Alternatively, consider using a Command Gateway.

If an application isn't directly interested in the outcome of a Command, the `dispatch(commandMessage)` method can be used.

SimpleCommandBus
----------------

The `SimpleCommandBus` is, as the name suggests, the simplest implementation. It does straightforward processing of commands in the thread that dispatches them. After a command is processed, the modified aggregate(s) are saved and generated events are published in that same thread. In most scenarios, such as web applications, this implementation will suit your needs. The `SimpleCommandBus` is the implementation used by default in the Configuration API.

Like most `CommandBus` implementations, the `SimpleCommandBus` allows interceptors to be configured. `CommandDispatchInterceptor`s are invoked when a command is dispatched on the Command Bus. The `CommandHandlerInterceptor`s are invoked before the actual command handler method is, allowing you to do modify or block the command. See [Command Interceptors](#command-interceptors) for more information.

Since all command processing is done in the same thread, this implementation is limited to the JVM's boundaries. The performance of this implementation is good, but not extraordinary. To cross JVM boundaries, or to get the most out of your CPU cycles, check out the other `CommandBus` implementations.

AsynchronousCommandBus
----------------------

As the name suggest, the `AsynchronousCommandBus` implementation executes Commands asynchronously from the thread that dispatches them. It uses an Executor to perform the actual handling logic on a different Thread.

By default, the `AsynchronousCommandBus` uses an unbounded cached thread pool. This means a thread is created when a Command is dispatched. Threads that have finished processing a Command are reused for new commands. Threads are stopped if they haven't processed a command for 60 seconds.

Alternatively, an `Executor` instance may be provided to configure a different threading strategy.

Note that the `AsynchronousCommandBus` should be shut down when stopping the application, to make sure any waiting threads are properly shut down. To shut down, call the `shutdown()` method. This will also shutdown any provided `Executor` instance, if it implements the `ExecutorService` interface.

DisruptorCommandBus
-------------------

The `SimpleCommandBus` has reasonable performance characteristics, especially when you've gone through the performance tips in [Performance Tuning](../part4/performance-tuning.md#performance-tuning). The fact that the `SimpleCommandBus` needs locking to prevent multiple threads from concurrently accessing the same aggregate causes processing overhead and lock contention.

The `DisruptorCommandBus` takes a different approach to multithreaded processing. Instead of having multiple threads each doing the same process, there are multiple threads, each taking care of a piece of the process. The `DisruptorCommandBus` uses the Disruptor (<http://lmax-exchange.github.io/disruptor/>), a small framework for concurrent programming, to achieve much better performance, by just taking a different approach to multi-threading. Instead of doing the processing in the calling thread, the tasks are handed off to two groups of threads, that each take care of a part of the processing. The first group of threads will execute the command handler, changing an aggregate's state. The second group will store and publish the events to the Event Store.

While the `DisruptorCommandBus` easily outperforms the `SimpleCommandBus` by a factor of 4(!), there are a few limitations:

-   The `DisruptorCommandBus` only supports Event Sourced Aggregates. This Command Bus also acts as a Repository for the aggregates processed by the Disruptor. To get a reference to the Repository, use `createRepository(AggregateFactory)`.

-   A Command can only result in a state change in a single aggregate instance.

-   When using a Cache, it allows only a single aggregate for a given identifier. This means it is not possible to have two aggregates of different types with the same identifier.

-   Commands should generally not cause a failure that requires a rollback of the Unit of Work. When a rollback occurs, the `DisruptorCommandBus` cannot guarantee that Commands are processed in the order they were dispatched. Furthermore, it requires a retry of a number of other commands, causing unnecessary computations.

-   When creating a new Aggregate Instance, commands updating that created instance may not all happen in the exact order as provided. Once the aggregate is created, all commands will be executed exactly in the order they were dispatched. To ensure the order, use a callback on the creating command to wait for the aggregate being created. It shouldn't take more than a few milliseconds.

To construct a `DisruptorCommandBus` instance, you need an `EventStore`. This component is explained in [Repositories and Event Stores](repositories-and-event-stores.md).

Optionally, you can provide a `DisruptorConfiguration` instance, which allows you to tweak the configuration to optimize performance for your specific environment:

-   Buffer size: The number of slots on the ring buffer to register incoming commands. Higher values may increase throughput, but also cause higher latency. Must always be a power of 2. Defaults to 4096.

-   ProducerType: Indicates whether the entries are produced by a single thread, or multiple. Defaults to multiple.

-   WaitStrategy: The strategy to use when the processor threads (the three threads taking care of the actual processing) need to wait for each other. The best WaitStrategy depends on the number of cores available in the machine, and the number of other processes running. If low latency is crucial, and the DisruptorCommandBus may claim cores for itself, you can use the `BusySpinWaitStrategy`. To make the Command Bus claim less of the CPU and allow other threads to do processing, use the `YieldingWaitStrategy`. Finally, you can use the `SleepingWaitStrategy` and `BlockingWaitStrategy` to allow other processes a fair share of CPU. The latter is suitable if the Command Bus is not expected to be processing full-time. Defaults to the `BlockingWaitStrategy`.

-   Executor: Sets the Executor that provides the Threads for the `DisruptorCommandBus`. This executor must be able to provide at least 4 threads. 3 of the threads are claimed by the processing components of the `DisruptorCommandBus`. Extra threads are used to invoke callbacks and to schedule retries in case an Aggregate's state is detected to be corrupt. Defaults to a `CachedThreadPool` that provides threads from a thread group called "DisruptorCommandBus".

-   TransactionManager: Defines the Transaction Manager that should ensure that the storage and publication of events are executed transactionally.

-   InvokerInterceptors: Defines the `CommandHandlerInterceptor`s that are to be used in the invocation process. This is the process that calls the actual Command Handler method.

-   PublisherInterceptors: Defines the `CommandHandlerInterceptor`s that are to be used in the publication process. This is the process that stores and publishes the generated events.

-   RollbackConfiguration: Defines on which Exceptions a Unit of Work should be rolled back. Defaults to a configuration that rolls back on unchecked exceptions.

-   RescheduleCommandsOnCorruptState: Indicates whether Commands that have been executed against an Aggregate that has been corrupted (e.g. because a Unit of Work was rolled back) should be rescheduled. If `false` the callback's `onFailure()` method will be invoked. If `true` (the default), the command will be rescheduled instead.

-   CoolingDownPeriod: Sets the number of seconds to wait to make sure all commands are processed. During the cooling down period, no new commands are accepted, but existing commands are processed, and rescheduled when necessary. The cooling down period ensures that threads are available for rescheduling the commands and calling callbacks. Defaults to 1000 (1 second).

-   Cache: Sets the cache that stores aggregate instances that have been reconstructed from the Event Store. The cache is used to store aggregate instances that are not in active use by the disruptor.

-   InvokerThreadCount: The number of threads to assign to the invocation of command handlers. A good starting point is half the number of cores in the machine.

-   PublisherThreadCount: The number of threads to use to publish events. A good starting point is half the number of cores, and could be increased if a lot of time is spent on IO.

-   SerializerThreadCount: The number of threads to use to pre-serialize events. This defaults to 1, but is ignored if no serializer is configured.

-   Serializer: The serializer to perform pre-serialization with. When a serializer is configured, the `DisruptorCommandBus` will wrap all generated events in a `SerializationAware` message. The serialized form of the payload and meta data is attached before they are published to the Event Store.

Command Interceptors
====================

One of the advantages of using a command bus is the ability to undertake action based on all incoming commands. Examples are logging or authentication, which you might want to do regardless of the type of command. This is done using Interceptors.

There are different types of interceptors: Dispatch Interceptors and Handler Interceptors. Dispatch Interceptors are invoked before a command is dispatched to a Command Handler. At that point, it may not even be sure that any handler exists for that command. Handler Interceptors are invoked just before the Command Handler is invoked.

Message Dispatch Interceptors
-----------------------------

Message Dispatch Interceptors are invoked when a command is dispatched on a Command Bus. They have the ability to alter the Command Message, by adding Meta Data, for example, or block the command by throwing an Exception. These interceptors are always invoked on the thread that dispatches the Command.

### Structural validation

There is no point in processing a command if it does not contain all required information in the correct format. In fact, a command that lacks information should be blocked as early as possible, preferably even before any transaction is started. Therefore, an interceptor should check all incoming commands for the availability of such information. This is called structural validation.

Axon Framework has support for JSR 303 Bean Validation based validation. This allows you to annotate the fields on commands with annotations like `@NotEmpty` and `@Pattern`. You need to include a JSR 303 implementation (such as Hibernate-Validator) on your classpath. Then, configure a `BeanValidationInterceptor` on your Command Bus, and it will automatically find and configure your validator implementation. While it uses sensible defaults, you can fine-tune it to your specific needs.

> **Tip**
>
> You want to spend as few resources on an invalid command as possible. Therefore, this interceptor is generally placed in the very front of the interceptor chain. In some cases, a Logging or Auditing interceptor might need to be placed in front, with the validating interceptor immediately following it.

The BeanValidationInterceptor also implements `MessageHandlerInterceptor`, allowing you to configure it as a Handler Interceptor as well.

Message Handler Interceptors
----------------------------

Message Handler Interceptors can take action both before and after command processing. Interceptors can even block command processing altogether, for example for security reasons.

Interceptors must implement the `MessageHandlerInterceptor` interface. This interface declares one method, `handle`, that takes three parameters: the command message, the current `UnitOfWork` and an `InterceptorChain`. The `InterceptorChain` is used to continue the dispatching process.

Unlike Dispatch Interceptors, Handler Interceptors are invoked in the context of the Command Handler. That means they can attach correlation data based on the Message being handled to the Unit of Work, for example. This correlation data will then be attached to messages being created in the context of that Unit of Work.

Handler Interceptors are also typically used to manage transactions around the handling of a command. To do so, register a `TransactionManagingInterceptor`, which in turn is configured with a `TransactionManager` to start and commit (or roll back) the actual transaction.

Distributing the Command Bus
============================

The CommandBus implementations described in earlier only allow Command Messages to be dispatched within a single JVM. Sometimes, you want multiple instances of Command Buses in different JVMs to act as one. Commands dispatched on one JVM's Command Bus should be seamlessly transported to a Command Handler in another JVM while sending back any results.

That's where the `DistributedCommandBus` comes in. Unlike the other `CommandBus` implementations, the `DistributedCommandBus` does not invoke any handlers at all. All it does is form a "bridge" between Command Bus implementations on different JVM's. Each instance of the `DistributedCommandBus` on each JVM is called a "Segment".

![Structure of the Distributed Command Bus](distributed-command-bus.png)

> **Note**
>
> While the distributed command bus itself is part of the Axon Framework Core module, it requires components that you can find in one of the *axon-distributed-commandbus-...* modules. If you use Maven, make sure you have the appropriate dependencies set. The groupId and version are identical to those of the Core module.

The `DistributedCommandBus` relies on two components: a `CommandBusConnector`, which implements the communication protocol between the JVM's, and the `CommandRouter`, which chooses a destination for each incoming Command. This Router defines which segment of the Distributed Command Bus should be given a Command, based on a Routing Key calculated by a Routing Strategy. Two commands with the same Routing Key will always be routed to the same segment, as long as there is no change in the number and configuration of the segments. Generally, the identifier of the targeted aggregate is used as a routing key.

Two implementations of the `RoutingStrategy` are provided: the `MetaDataRoutingStrategy`, which uses a Meta Data property in the Command Message to find the routing key, and the `AnnotationRoutingStrategy`, which uses the `@TargetAggregateIdentifier` annotation on the Command Messages payload to extract the Routing Key. Obviously, you can also provide your own implementation.

By default, the RoutingStrategy implementations will throw an exception when no key can be resolved from a Command Message. This behavior can be altered by providing a UnresolvedRoutingKeyPolicy in the constructor of the MetaDataRoutingStrategy or AnnotationRoutingStrategy. There are three possible policies:

-   ERROR: This is the default, and will cause an exception to be thrown when a Routing Key is not available

-   RANDOM\_KEY: Will return a random value when a Routing Key cannot be resolved from the Command Message. This effectively means that those commands will be routed to a random segment of the Command Bus.

-   STATIC\_KEY: Will return a static key (being "unresolved") for unresolved Routing Keys. This effectively means that all those commands will be routed to the same segment, as long as the configuration of segments does not change.

JGroupsConnector
----------------

The `JGroupsConnector` uses (as the name already gives away) JGroups as the underlying discovery and dispatching mechanism. Describing the feature set of JGroups is a bit too much for this reference guide, so please refer to the [JGroups User Guide](http://www.jgroups.org/ug.html) for more details.

Since JGroups handles both discovery of nodes and the communication between them, the `JGroupsConnector` acts as both a `CommandBusConnector` and a `CommandRouter`.

> **Note**
> 
> You can find the JGroups specific components for the `DistributedCommandBus` in the `axon-distributed-commandbus-jgroups` module.

The JGroupsConnector has four mandatory configuration elements:

-   The first is a JChannel, which defines the JGroups protocol stack. Generally, a JChannel is constructed with a reference to a JGroups configuration file. JGroups comes with a number of default configurations which can be used as a basis for your own configuration. Do keep in mind that IP Multicast generally doesn't work in Cloud Services, like Amazon. TCP Gossip is generally a good start in such type of environment.

-   The Cluster Name defines the name of the Cluster that each segment should register to. Segments with the same Cluster Name will eventually detect each other and dispatch Commands among each other.

-   A "local segment" is the Command Bus implementation that dispatches Commands destined for the local JVM. These commands may have been dispatched by instances on other JVMs or from the local one.

-   Finally, the Serializer is used to serialize command messages before they are sent over the wire.

> **Note**
>
> When using a Cache, it should be cleared out when the `ConsistentHash` changes to avoid potential data corruption (e.g. when commands don't specify a `@TargetAggregateVersion` and a new member quickly joins and leaves the JGroup, modifying the aggregate while it's still cached elsewhere.)

Ultimately, the JGroupsConnector needs to actually connect, in order to dispatch Messages to other segments. To do so, call the `connect()` method. 

``` java
JChannel channel = new JChannel("path/to/channel/config.xml");
CommandBus localSegment = new SimpleCommandBus();
Serializer serializer = new XStreamSerializer();

JGroupsConnector connector = new JGroupsConnector(channel, "myCommandBus", localSegment, serializer);
DistributedCommandBus commandBus = new DistributedCommandBus(connector, connector);

// on one node:
commandBus.subscribe(CommandType.class.getName(), handler);
connector.connect();

// on another node, with more CPU:
commandBus.subscribe(CommandType.class.getName(), handler);
commandBus.subscribe(AnotherCommandType.class.getName(), handler2);
commandBus.updateLoadFactor(150); // defaults to 100
connector.connect();

// from now on, just deal with commandBus as if it is local...
```

> **Note**
>
> Note that it is not required that all segments have Command Handlers for the same type of Commands. You may use different segments for different Command Types altogether. The Distributed Command Bus will always choose a node to dispatch a Command to that has support for that specific type of Command.

If you use Spring, you may want to consider using the `JGroupsConnectorFactoryBean`. It automatically connects the Connector when the ApplicationContext is started, and does a proper disconnect when the `ApplicationContext` is shut down. Furthermore, it uses sensible defaults for a testing environment (but should not be considered production ready) and autowiring for the configuration.

Spring Cloud Connector
----------------------

The Spring Cloud Connector setup uses the service registration and discovery mechanism described by [Spring Cloud](http://projects.spring.io/spring-cloud/) for distributing the Command Bus. You are thus left free to choose which Spring Cloud implementation to use to distribute your commands. An example implementations is the Eureka Discovery/Eureka Server combination.
 
 > **Note**
 >
 > The `SpringCloudCommandRouter` uses the Spring Cloud specific `ServiceInstance.Metadata` field to inform all the nodes in the system of its message routing information. It is thus of importance that the Spring Cloud implementation selected supports the usage of the `ServiceInstance.Metadata` field. If the desired Spring Cloud implementation does not support the modification of the `ServiceInstance.Metadata` (e.g. Consul), the `SpringCloudHttpBackupCommandRouter` is a viable solution. See the end of this chapter for configuration specifics on the `SpringCloudHttpBackupCommandRouter`. 

Giving a description of every Spring Cloud implementation would push this reference guide to far. Hence we refer to their respective documentations for further information.

The Spring Cloud Connector setup is a combination of the `SpringCloudCommandRouter` and a `SpringHttpCommandBusConnector`, which respectively fill the place of the `CommandRouter` and the `CommandBusConnector` for the `DistributedCommandBus`.

> **Note**
>
> When using the `SpringCloudCommandRouter`, make sure that your Spring application is has heartbeat events enabled. The implementation leverages the heartbeat events published by a Spring Cloud application to check whether its knowledge of all the others nodes is up to date. Hence if heartbeat events are disabled the majority of the Axon applications within your cluster will not be aware of the entire set up, thus posing issues for correct command routing.

The `SpringCloudCommandRouter` has to be created by providing the following:

- A "discovery client" of type `DiscoveryClient`. This can be provided by annotating your Spring Boot application with `@EnableDiscoveryClient`, which will look for a Spring Cloud implementation on your classpath.
 
- A "routing strategy" of type `RoutingStrategy`. The `axon-core` module currently provides several implementations, but a function call can suffice as well. If you want to route the Commands based on the 'aggregate identifier' for example, you would use the `AnnotationRoutingStrategy` and annotate the field on the payload that identifies the aggregate with `@TargetAggregateIdentifier`.

Other optional parameters for the `SpringCloudCommandRouter`  are:

- A "service instance filter" of type `Predicate<ServiceInstance>`. This predicate is used to filter out `ServiceInstances` which the `DiscoveryClient` might encounter which by forehand you know will not handle any command messages. This might be useful if you've got several services within the Spring Cloud Discovery Service set up which you do not want to take into account for command handling, ever.  

- A "consistent hash change listener" of type `ConsistentHashChangeListener`. Adding a consistent hash change listener provides you the opportunity to perform a specific task if  new members have been added to the known command handlers set.

The `SpringHttpCommandBusConnector` requires three parameters for creation:
 
- A "local command bus" of type `CommandBus`. This is the Command Bus implementation that dispatches Commands to the local JVM. These commands may have been dispatched by instances on other JVMs or from the local one.

- A `RestOperations` object to perform the posting of a Command Message to another instance.

- Lastly a "serializer" of type `Serializer`. The serializer is used to serialize the command messages before they are sent over the wire.

> **Note**
>
> The Spring Cloud Connector specific components for the `DistributedCommandBus` can be found in the `axon-distributed-commandbus-springcloud` module.

The `SpringCloudCommandRouter` and `SpringHttpCommandBusConnector` should then both be used for creating the `DistributedCommandsBus`. In Spring Java config, that would look as follows:

```java
// Simple Spring Boot App providing the `DiscoveryClient` bean
@EnableDiscoveryClient
@SpringBootApplication
public class MyApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    // Example function providing a Spring Cloud Connector
    @Bean
    public CommandRouter springCloudCommandRouter(DiscoveryClient discoveryClient) {
        return new SpringCloudCommandRouter(discoveryClient, new AnnotationRoutingStrategy());
    }
    
    @Bean
    public CommandBusConnector springHttpCommandBusConnector(@Qualifier("localSegment") CommandBus localSegment,
                                                             RestOperations restOperations,
                                                             Serializer serializer) {
        return new SpringHttpCommandBusConnector(localSegment, restOperations, serializer);
    }
    
    @Primary // to make sure this CommandBus implementation is used for autowiring
    @Bean
    public DistributedCommandBus springCloudDistributedCommandBus(CommandRouter commandRouter, 
                                                                  CommandBusConnector commandBusConnector) {
        return new DistributedCommandBus(commandRouter, commandBusConnector);
    }

}
```
``` java
// if you don't use Spring Boot Autoconfiguration, you will need to explicitly define the local segment:
@Bean
@Qualifier("localSegment")
public CommandBus localSegment() {
    return new SimpleCommandBus();
}

```
> **Note**
>
> Note that it is not required that all segments have Command Handlers for the same type of Commands. You may use different segments for different Command Types altogether. The Distributed Command Bus will always choose a node to dispatch a Command to that has support for that specific type of Command.

##### Spring Cloud Http Back Up Command Router

Internally, the `SpringCloudCommandRouter` uses the `Metadata` map contained in the Spring Cloud `ServiceInstance` to communicate the allowed message routing information throughout the distributed Axon environment. If the desired Spring Cloud implementation however does not allow the modification of the `ServiceInstance.Metadata` field (e.g. Consul), one can choose to instantiate a `SpringCloudHttpBackupCommandRouter` instead of the `SpringCloudCommandRouter`. 

The `SpringCloudHttpBackupCommandRouter`, as the name suggests, has a back up mechanism if the `ServiceInstance.Metadata` field does not contained the expected message routing information. That back up mechanism is to provide an HTTP endpoint from which the message routing information can be retrieved and by simultaneously adding the functionality to query that endpoint of other known nodes in the cluster to retrieve their message routing information. As such the back up mechanism functions is a Spring Controller to receive requests at a specifiable endpoint and uses a `RestTemplate` to send request to other nodes at the specifiable endpoint.

To use the `SpringCloudHttpBackupCommandRouter` instead of the `SpringCloudCommandRouter`, add the following Spring Java configuration (which replaces the `SpringCloudCommandRouter` method in our earlier example):

```java
@Configuration
public class MyApplicationConfiguration {
    @Bean
    public CommandRouter springCloudHttpBackupCommandRouter(DiscoveryClient discoveryClient, 
                                                            RestTemplate restTemplate, 
                                                            @Value("${axon.distributed.spring-cloud.fallback-url}") String messageRoutingInformationEndpoint) {
        return new SpringCloudHttpBackupCommandRouter(discoveryClient, new AnnotationRoutingStrategy(), restTemplate, messageRoutingInformationEndpoint);
    }
}
``` 
