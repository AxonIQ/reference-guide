# JobRunrPro

The purpose of this extension is to leverage some features only available in the Pro version of [JobRunr](https://www.jobrunr.io/en/documentation/pro/).
Only the Pro version allows to search existing jobs by status and label. This is required for the deadline manager to implement the `cancelAll` methods.
The [deadline managers](../axon-framework/deadlines/deadline-managers.md) section has more in depth information on deadline managers.
Although jobs created with the non-pro `DeadlineManager` will be eligible to be canceled, this is only true when they were created with the `4.8` or later versions.
Jobs created with the `4.7` version are missing the correct labels and will not be found when trying to cancel them.

## Spring usage

For spring usage, be sure to include the starters, both of JobRunr Pro and the extension. The deadline manager should be available by autowiring.
It can by used in aggregates and sagas using something like:
```java
@Autowired
void setDeadlineManager(DeadlineManager deadlineManager) {
    this.deadlineManager = deadlineManager;
}
```

## Non Spring Usage

An `JobRunrProDeadlineManager` instance can be created using the builder like this:
```java
JobRunrProDeadlineManager.proBuilder()
        .jobScheduler(jobScheduler)
        .storageProvider(storageProvider)
        .scopeAwareProvider(scopeAwareProvider)
        .serializer(serializer)
        .transactionManager(transactionManager)
        .spanFactory(spanFactory)
        .build();
```

You probably want to use some form of dependency injection instead or creating a new deadline manager each time you need one.



