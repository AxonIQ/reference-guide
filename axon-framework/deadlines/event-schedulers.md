# Event Schedulers

This section will proceed with a suggested course of action when utilizing the `EventScheduler` for dealing with deadlines.

To help understand this better lets take the scenario of a saga: 
It is easy to make a saga take action when something happens. 
After all, there is an event to notify the saga. But what if you want your saga to do something when _nothing_ happens? 
That's what deadlines are used for. 
For invoices, that is typically several weeks, whereas the confirmation of a credit card payment should occur within a few seconds.

## Scheduled Events as Deadlines

In Axon, you can use an `EventScheduler` to schedule an event for publication. In the example of an invoice, you would expect the invoice to be paid within thirty days. A saga would, after sending the `CreateInvoiceCommand`, schedule an `InvoicePaymentDeadlineExpiredEvent` to be published in 30 days. The `EventScheduler` returns a `ScheduleToken` after scheduling an event. This token can be used to cancel the schedule, for example when a payment of an Invoice has been received.

Axon provides four `EventScheduler` implementations:

 1. Pure Java
 2. [JobRunr](https://www.jobrunr.io/) based
 3. [Quartz](http://www.quartz-scheduler.org/) based
 4. [Axon Server](../../axon-server/introduction.md) based

The pure-Java implementation of the `EventScheduler` uses a `ScheduledExecutorService` to schedule event publication. 
Although the timing of this scheduler is very reliable, it is a pure in-memory implementation. 
Once the JVM is shut down, all schedules are lost. 
This makes this implementation unsuitable for long-term schedules.
The `SimpleEventScheduler` needs to be configured with an `EventBus` and a `SchedulingExecutorService` \(see the static methods on the `java.util.concurrent.Executors` class for helper methods\).

The `JonRunrEventScheduler` is a more reliable and enterprise-worthy implementation.
It offers several ways to persist the scheduled jobs, and has good integration with Spring Boot.
Using JobRunr as underlying scheduling mechanism, it provides more powerful features, such as clustering, misfire management as well as an optional dashboard.
This means event publication is guaranteed. It might be a little late, but it will be published.
It needs to be configured with a JobRunr `JobScheduler`, an `EventBus` and a `Serializer`.

The `QuartzEventScheduler` is an alternative enterprise-worthy implementation, but the project has not much recent activity. 
Using Quartz as underlying scheduling mechanism, it provides features, such as persistence, clustering and misfire management.
It needs to be configured with a Quartz `Scheduler` and an `EventBus`. 
Optionally, you may set the name of the group that Quartz jobs are scheduled in, which defaults to `"AxonFramework-Events"`.

The `AxonServerEventScheduler` uses Axon Server to schedule events for publication.
As such, it is a *hard requirement* to use Axon Server as your Event Store solution to utilize this event scheduler.
Just as the `QuartzEventScheduler`, the `AxonServerEventScheduler` is a reliable and enterprise-worthy implementation of the `EventScheduler` interface.
Creating a `AxonServerEventScheduler` can be done through its builder, whose sole requirement is the `AxonServerConnectionManager`.

It is important to note that the `JubRunrEventScheduler`, `QuartzEventScheduler` and `AxonServerEventScheduler` all should use the [event `Serializer`](../serialization.md#event-serialization) to serialize and deserialize the scheduled event.
If the `Serializer` used by the scheduler does not align with the `Seralizer` used by the event store, exceptional scenarios should be expected.
The Quartz implementation's `Serializer` can be set by defining a different `EventJobDataBinder`, whereas the JobRunr and Axon Server implementations allows defining the used `Serializer` directly through the builder.

One or more components will be listening for scheduled Events. 
These components might rely on a transaction bound to the thread that invokes them. 
Scheduled events are published by threads managed by the `EventScheduler`. 
To manage transactions on these threads, you can configure a `TransactionManager` or a `UnitOfWorkFactory` that creates a transaction bound unit of work.

> **Spring Configuration**
>
> Spring users can use the `QuartzEventSchedulerFactoryBean` or `SimpleEventSchedulerFactoryBean` for easier configuration. 
> It allows you to set the `PlatformTransactionManager` directly.
>
> Spring Boot users which rely on Axon Server do not have to define anything.
> The auto configuration will automatically create a `AxonServerEventScheduler` for them.
> 
> Spring Boot users which want to use the JobRunr event scheduler can add [`jobrunr-spring-boot-starter`](https://mvnrepository.com/artifact/org.jobrunr/jobrunr-spring-boot-starter) as dependency.
> In addition adding a `EventScheduler` bean configuration needs to be added. This should look like:
> ```java
> @Bean
> public EventScheduler eventScheduler(
>         @Qualifier("eventSerializer") final Serializer serializer,
>         final JobScheduler jobScheduler,
>         final EventBus eventBus,
>         final TransactionManager transactionManager,
> ) {
>     return JobRunrEventScheduler.builder()
>             .jobScheduler(jobScheduler)
>             .serializer(serializer)
>             .eventBus(eventBus)
>             .transactionManager(transactionManager)
>             .build();
> }
> ```

