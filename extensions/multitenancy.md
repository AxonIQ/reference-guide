# Multitenancy Extension

The Axon Framework Multitenancy Extension provides your application with the ability to serve multiple tenants (event-stores) at once.
Multi-tenancy is important in cloud computing, as this extension provides the ability to connect tenants dynamically, physical separate tenant-data, and scale tenants independently.

### Requirements

- Currently, It's possible to configure extension using **Axon Framework 4.6+** together with **Spring Framework**.
- Minimal configuration and out-of-the box solution is available only for **Axon Server EE 4.6+** or Axon Cloud (*).
- Any other custom user solutions should implement [own factory beans for components and tenant provider](https://github.com/AxonFramework/extension-multitenancy/blob/main/multitenancy-spring-boot-autoconfigure/src/main/java/org/axonframework/extensions/multitenancy/autoconfig/MultiTenancyAxonServerAutoConfiguration.java)
- If you wish to enable multi-tenancy for your projections and token store, note that only JPA is supported out-of-the box.

> **Axon Cloud and the Multi-Tenancy extension**
>
> Currently, Axon Cloud works only with static tenant configuration.

### Configuration

A minimal configuration is needed to get this extension up and running.
Please choose **either** the static or the dynamic tenant configuration.

#### Static tenants configuration

If you have a predefined list of tenants that your application should connect to, set following property:
`axon.axonserver.contexts=tenant-context-1,tenant-context-2,tenant-context-3`

#### Dynamic tenants configuration

If you plan to create tenants during runtime, you can define a predicate that will tell the application to which tenant-contexts to connect to once they appear:

```java
@Bean
public TenantConnectPredicate tenantFilterPredicate() {
    return context -> context.tenantId().startsWith("tenant-");
}
```

### Route Messages to specific tenants

Backbone of multitenancy is ability to route message to specific tenant.
This extension offers you meta-data based routing which is ready to be used with minimal configuration.
Also, one may wish to define stronger contract and include tenant information in message payload, which is also possible by defining custom tenant resolver.
        
#### Using meta-data

By default, to route any `Message` to a specific tenant, you need to tag the initial message that enters your system with metadata.
This is done with a meta-data helper function, which should add the tenant name with key `TenantConfiguration.TENANT_CORRELATION_KEY`.

```java
message.andMetaData(Collections.singletonMap(TENANT_CORRELATION_KEY, "tenant-context-1")
```

Note that you only need to add metadata to the initial message entering your system. 
Any message produced as a consequence of the initial message will have this metadata copied automatically using a `CorrelationDataProvider`.

#### Custom tenant resolver

If you wish to define a custom tenant resolver, set following property:

`axon.multi-tenancy.use-metadata-helper=false`

Then define the custom tenant resolver bean. 
The following example can use the message payload to route a message to specific tenant:

```java
@Bean
public TargetTenantResolver<Message<?>> customTargetTenantResolver() {
    return (message, tenants) ->
            TenantDescriptor.tenantWithId(
                    ((TenantAwareMessage) message.getPayload()).getTenantName()
            );
}
```

In example above, all messages should implement custom `TenantAwareMessage` interface that exposes tenant name.
Then we can use this interface to extract tenant name from the payload and define our tenant resolver.

### Multi-tenant projections

If you wish to use distinct tenant-databases to store projections and tokens, please configure the following:

```java
@Bean
public Function<TenantDescriptor, DataSourceProperties> tenantDataSourceResolver() {
    return tenant -> {
        DataSourceProperties properties = new DataSourceProperties();
        properties.setUrl("jdbc:postgresql://localhost:5432/"+tenant.tenantId());
        properties.setDriverClassName("org.postgresql.Driver");
        properties.setUsername("postgres");
        properties.setPassword("postgres");
        return properties;
    };
}
```

Note that this works by using the JPA multi-tenancy support provided in this extension.
That means that currently only SQL Databases are supported out of the box.

If you wish to implement multi-tenancy for a different type of databases (e.g. NoSQL) make sure that your projection database supports multi-tenancy, too.
When doing so, you can find it which tenants own the transaction by invoking `TenantWrappedTransactionManager.getCurrentTenant()`.

> **Pre-initialized schema**
>
> Schema migration tools like Liquibase or Flyway usually won't be able to initialize schemas for dynamically created data sources.
> Hence, any data source that you use needs to have a the schema pre-initialized.
 
#### Resetting projections

Resetting projections works a bit different, because there are multiple instances of the "same" event processor.
Namely, one per tenant.

Regard the following sample to reset an Event Processor for a specific tenant:

```java
TrackingEventProcessor trackingEventProcessor = configuration.eventProcessingConfiguration()
                                                             .eventProcessor("com.demo.query-ep@tenant-context-1", TrackingEventProcessor.class)
                                                             .get();
```

Note that the convention for naming tenant-specific event processor is `{even processor name}@{tenant name}`.

If you need to access all tenant event processors in one go, you can retrieve the `MultiTenantEventProcessor` for a specific processing name.
The `MultiTenantEventProcessor` acts as a proxy event processor referencing all tenant-specific event processors.

### Supported multi-tenant components

Currently, the following infrastructure components support multi-tenancy:

- <span style="color:green">MultiTenantCommandBus</span>
- <span style="color:green">MultiTenantEventProcessor</span>
- <span style="color:green">MultiTenantEventStore</span>
- <span style="color:green">MultiTenantQueryBus</span>
- <span style="color:green">MultiTenantQueryUpdateEmitter</span>
- <span style="color:green">MultiTenantEventProcessorControlService</span>
- <span style="color:green">MultiTenantDataSourceManager</span>

The following components are not yet supported:

- <span style="color:red">MultitenantDeadlineManager</span>
- <span style="color:red">MultitenantEventScheduler</span>


### Disabling this Extension

By default, this extension is enabled if found on class path when utilizing Spring Boot.
If you wish to disable the extension without removing the dependency, you can set the following property to `false`:

`axon.multi-tenancy.enabled=false`
