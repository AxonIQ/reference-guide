# Aggregate Polymorphism

In certain cases it is beneficial to have a polymorphic hierarchy in aggregate structure. Subtypes in polymorphic 
aggregate hierarchy inherit `@CommandHandler`s, `@EventSourcingHandler`s and `@CommandHandlerInterceptor`s from the 
super aggregates. Based on `@AggregateIdentifier` the correct aggregate type is loaded and command is executed on it. 
Let's take a look at the following example:

```java
public class Parent {}

public class AggregateA extends Parent {}

public class AggregateB extends AggregateA {}

public class AggregateC extends AggregateA {}

public class AggregateD extends AggregateB {}
```

We can define this structure as Polymorphic Aggregate of type `AggregateA` and subtypes of `AggregateB`, `AggregateC`, 
and `AggregateD`. If there handlers present on `Parent` class, those will be present on all aggregates as well. 

While modeling a polymorphic aggregate hierarchy it is important to keep these constraints in mind:
- It is not allowed to have a constructor annotated with `@CommandHandler` on abstract aggregate. The rationale for this 
is that an abstract aggregate can never be created.
- Having creational command handlers of the same command name on different aggregates in the same hierarchy is forbidden 
too, since Axon cannot derive which one to invoke. 
- In polymorphic aggregate hierarchy it is allowed to have only one `@AggregateIdentifier` and one `@AggregateVersion` 
field.

## Registering aggregate subtypes

Polymorphic aggregate hierarchy can be registered via `AggregateConfigurer` by invoking `registerSubtype`. Do note that 
children of top aggregate that are not registered as a subtype will automatically be registered as a subtype. In the 
following example `AggregateB` is transitively registered as a subtype of `AggregateA`. However, if there is an 
`AggregateE extends AggregateD` defined, it will not be picked up (unless explicitly registered as a subtype).

```java
AggregateConfigurer<AggregateA> configurer = AggregateConfigurer.defaultConfiguration(AggregateA.class);
configurer.registerSubtype(AggregateC.class);
configurer.registerSubtype(AggregateD.class);
```

> If you are using Spring, polymorphic hierarchy will be automatically detected based on `@Aggregate` annotations and 
class hierarchy.
