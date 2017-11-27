Configuration API
=================

Axon keeps a strict separation when it comes to business logic and infrastructure configuration. In order to do so, Axon will provide a number of building blocks that take care of the infrastructural concerns, such as transaction management around a message handler. The actual payload of the messages and the contents of the handler are implemented in (as much as possible) Axon-independent Java classes.

To make the configuration of these infrastructure components easier and to define their relationship with each of the functional components, Axon provides a Configuration API.

Setting up a configuration
--------------------------

Getting a default configuration is very easy:

``` java
Configuration config = DefaultConfigurer.defaultConfiguration()
                                        .buildConfiguration();
```

This configuration provides the building blocks for dispatching Messages using implementations that handle messages on the threads that dispatch them.

Obviously, this configuration would not be very useful. You would have to register your Command Model objects and Event Handlers to this configuration to be useful.

To do so, use the `Configurer` instance returned by the `.defaultConfiguration()` method.

``` java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
```

The configurer provides a multitude of methods that allow you to register these components. How to configure those is described in detail in each component's respective chapter.

The general form in which components are registered, is the following:

``` java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.registerCommandHandler(c -> doCreateComponent());
```

Note the lambda expression in the `registerCommandBus` invocation. The `c` parameter of this expression is the configuration object that described the complete configuration. If your component requires any other components to function properly, it can use this configuration to retrieve them.

For example, to register a Command Handler that requires a serializer:

``` java
configurer.registerCommandHandler(c -> new MyCommandHandler(c.serializer());
```

Not all components have their explicit accessor method. To retrieve a component from the configuration, use:
``` java
configurer.registerCommandHandler(c -> new MyCommandHandler(c.getComponent(MyOtherComponent.class));
```

This component must be registered with the Configurer, using `configurer.registerComponent(componentType, builderFunction)`. The builder function will receive the `Configuration` object as input parameter.

Setting up a configuration using Spring
---------------------------------------

When using Spring, there is no need to explicitly use the `Configurer`. Instead, you can simply put the `@EnableAxon` on one of your Spring `@Configuration` classes.

Axon will use the Spring Application Context to locate specific implementations of building blocks and provide default for those that are not there. So, instead of registering the building blocks with the `Configurer`, in Spring you just have to make them available in the Application Context as `@Bean`s.
