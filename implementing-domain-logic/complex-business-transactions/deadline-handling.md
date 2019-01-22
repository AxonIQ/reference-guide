# Deadline handling

It is easy to make a saga take action when something happens. After all, there is an event to notify the saga. But what if you want your saga to do something when _nothing_ happens? That's what deadlines are used for. In invoices, that is typically several weeks, while the confirmation of a credit card payment should occur within a few seconds.

In Axon, you can use an `EventScheduler` to schedule an event for publication. In the example of an Invoice, you would expect that invoice to be paid within thirty days. A saga would, after sending the `CreateInvoiceCommand`, schedule an `InvoicePaymentDeadlineExpiredEvent` to be published in 30 days. The `EventScheduler` returns a `ScheduleToken` after scheduling an èvent. This token can be used to cancel the schedule, for example when a payment of an Invoice has been received.

Axon provides two `EventScheduler` implementations: a pure Java one and one using Quartz 2 as a backing scheduling mechanism.

This pure-Java implementation of the `EventScheduler` uses a `ScheduledExecutorService` to schedule event publication. Although the timing of this scheduler is very reliable, it is a pure in-memory implementation. Once the JVM is shut down, all schedules are lost. This makes this implementation unsuitable for long-term schedules.

The `SimpleEventScheduler` needs to be configured with an `EventBus` and a `SchedulingExecutorService` \(see the static methods on the `java.util.concurrent.Executors` class for helper methods\).

The `QuartzEventScheduler` is a more reliable and enterprise-worthy implementation. Using Quartz as underlying scheduling mechanism, it provides more powerful features, such as persistence, clustering and misfire management. This means event publication is guaranteed. It might be a little late, but it will be published.

It needs to be configured with a Quartz `Scheduler` and an `EventBus`. Optionally, you may set the name of the group that Quartz jobs are scheduled in, which defaults to `"AxonFramework-Events"`.

One or more components will be listening for scheduled Events. These components might rely on a Transaction being bound to the thread that invokes them. Scheduled events are published by threads managed by the `EventScheduler`. To manage transactions on these threads, you can configure a `TransactionManager` or a `UnitOfWorkFactory` that creates a transaction bound unit of work.

> **Note**
>
> Spring users can use the `QuartzEventSchedulerFactoryBean` or `SimpleEventSchedulerFactoryBean` for easier configuration. It allows you to set the `PlatformTransactionManager` directly.
