#Multitenancy Extension

Axon Framework Multitenancy Extension provides your application ability to serve multiple tenants (event-stores) at once.
Multi-tenancy is important in cloud computing and this extension will provide ability to connect to tenant dynamically, physical separate tenants data, scale tenants independently...


### Requirements

Following requirements needs to be meet for extension to work:
- Use **Spring Framework** together with **Axon Framework 4.6+**
- Use **Axon Server EE 4.6+** or Axon Cloud as event store (*)
- This is not hard requirement but if you wish to enable multitenancy on projection side, only JPA is supported out-of-the box

> ** Axon Cloud **
>
> Axon Cloud works only with static tenant configuration.

### Configuration

Minimal configuration is needed to get extension up and running.
Choose to use **either** static or dynamic tenant configuration.

#### Static tenants configuration

If you have predefined list of contexts that your application should connect to, set following property:
`axon.axonserver.contexts=tenant-context-1,tenant-context-2,tenant-context-3`

#### Dynamic tenants configuration

If you plan to create tenants in runtime, you can define a predicate which will tell application to which contexts to connect to once they appear in runtime:

```java
    @Bean
public TenantConnectPredicate tenantFilterPredicate() {
        return context -> context.tenantId().startsWith("tenant-");
        }
```

### Route message to specific tenant

#### Using meta-data

By default, to route message to specific tenant you need to tag initial message that enters your system with metadata.
This is done with meta-data helper, and to route message to specific tenant you should set tenant name to metadata with key `TenantConfiguration.TENANT_CORRELATION_KEY`.

```java
message.andMetaData(Collections.singletonMap(TENANT_CORRELATION_KEY, "tenant-context-1")
```

Metadata needs to be added only to initial message that enters your system. Any message that is produced by consequence of initial message will have this metadata copied automatically using to `CorrelationProvider`.

#### Custom tenant resolver

If you wish to define custom tenant resolver set following property:

`axon.multi-tenancy.use-metadata-helper=false`

Then define custom tenant resolver bean. For example following bean can use message payload to route message to specific tenant:

```java
    @Bean
    public TargetTenantResolver<Message<?>> customTargetTenantResolver() {
        return (message, tenants) ->
                TenantDescriptor.tenantWithId(
                        ((TenantTaggedMessage) message.getPayload()).getTenantName()
                );
    }
```

### Multi-tenant projections

If you wish to use distinct database to store projections and token store for each tenant, configure following bean:

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

Note that this works by using JPA multi-tenancy support, that means only SQL Databases are supported out of the box.
If you wish to implement multi-tenancy for a different type of databases (e.g. NoSQL) make sure that your projection database supports multi-tenancy.
While in transaction you may find out which tenant owns transaction by calling:` TenantWrappedTransactionManager.getCurrentTenant()`.

> **Pre initialize schema**
>
> Schema migration tools like Liquibase or Flyway usually won't be able to initialise schemas for dynamically created data sources.
> Any datasource that will use needs to have pre-initialized schema.
 
#### Resetting projections

Resetting projections works a bit different because there are multiple instances of "same" event processor group (one per each tenant).

Reset specific tenant event processor group:

```java
    TrackingEventProcessor trackingEventProcessor = configuration.eventProcessingConfiguration()
                                                                     .eventProcessor("com.demo.query-ep@tenant-context-1",
                                                                                     TrackingEventProcessor.class)
                                                                     .get();
```

Convention for naming event processor is: `{even processor name}@{tenant name}`

Access all tenant event processors by retrieving `MultiTenantEventProcessor` only.
`MultiTenantEventProcessor` acts as a proxy Event Processor that references all tenant event processors.

### Supported multi-tenant components

Currently, supported multi-tenants components are as follows:

- <span style="color:green">MultiTenantCommandBus</span>
- <span style="color:green">MultiTenantEventProcessor</span>
- <span style="color:green">MultiTenantEventStore</span>
- <span style="color:green">MultiTenantQueryBus</span>
- <span style="color:green">MultiTenantQueryUpdateEmitter</span>
- <span style="color:green">MultiTenantEventProcessorControlService</span>
- <span style="color:green">MultiTenantDataSourceManager</span>

Not yet supported multi-tenants components are:

- <span style="color:red">MultitenantDeadlineManager</span>
- <span style="color:red">MultitenantEventScheduler</span>


### Disable extension

By default, extension is automatically enabled if found on class path.
If you wish to disable extension without removing extension use following property

`axon.multi-tenancy.enabled=false`
