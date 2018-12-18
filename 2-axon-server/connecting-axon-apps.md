# Connecting Axon Applications

Axon Server is configured by default when you use the `axon-spring-boot-starter` with Maven dependency:

```markup
<dependency>
  <groupId>org.axonframework</groupId>
  <artifactId>axon-spring-boot-starter</artifactId>
  <version>{version}</version>
</dependency>
```

The same configuration can also be done manually, when you are not using Spring boot. For this you need to include the following dependency:

```markup
<dependency>
  <groupId>org.axonframework</groupId>
  <artifactId>axonserver-connector</artifactId>
  <version>{version}</version>
</dependency>
```

In your Axon application you need to configure the various buses manually, e.g.:

```java
public class AxonStarter {
    public static void main(String[] args) {
        AxonServerConfiguration axonServerConfiguration = 
            AxonServerConfiguration.newBuilder("localhost", "AxonStarter").build();
        PlatformConnectionManager platformConnectionManager = 
            new PlatformConnectionManager(axonServerConfiguration);

        Serializer genericSerializer = new JacksonSerializer();
        
        EventBus axonHubEventStore = new AxonServerEventStore(
                                            axonServerConfiguration, 
                                            platformConnectionManager, 
                                            genericSerializer, 
                                            genericSerializer,
                                            NoOpEventUpcaster.INSTANCE);
                                            
        CommandBus localSegment = new SimpleCommandBus();       
        CommandBus axonHubCommandBus = new AxonServerCommandBus(
                                            platformConnectionManager, 
                                            axonServerConfiguration, 
                                            localSegment, 
                                            genericSerializer,
                                            new AnnotationRoutingStrategy());

        SimpleQueryBus localQueryBus = new SimpleQueryBus();
        QueryBus axonHubQueryBus = new AxonServerQueryBus(platformConnectionManager, 
                                              axonServerConfiguration, 
                                              localQueryBus, 
                                              localQueryBus,  
                                              genericSerializer,
                                              genericSerializer,
                                              new QueryPriorityCalculator() {});

        Configuration config = 
            DefaultConfigurer.defaultConfiguration()
                                .configureEventBus(c -> axonHubEventStore)
                                .configureCommandBus(c -> axonHubCommandBus)
                                .configureQueryBus(c -> axonHubQueryBus)
                                // more configuration for Axon
                                .configureSerializer(c -> genericSerializer)
                                .buildConfiguration();

        config.start();
    }
}
```

