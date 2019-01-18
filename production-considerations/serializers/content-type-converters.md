# ContentTypeConverters

An [upcaster](../application-versioning/versioning-events.md#event-upcasting) works on a given content type \(e.g. dom4j Document\). 
To provide extra flexibility between upcasters, content types between chained upcasters may vary. 
Axon will try to convert between the content types automatically by using `ContentTypeConverter`s. 
It will search for the shortest path from type `x` to type `y`, perform the conversion and pass the converted value into the requested upcaster. 
For performance reasons, conversion will only be performed if the `canUpcast` method on the receiving upcaster yields true.

The `ContentTypeConverter`s may depend on the type of serializer used. 
Attempting to convert a `byte[]` to a dom4j `Document` will not make any sense unless a `Serializer` was used that writes an event as XML. 
To make sure the `UpcasterChain` has access to the serializer-specific `ContentTypeConverter`s,
 you can pass a reference to the serializer to the constructor of the `UpcasterChain`.

> **Tip**
>
> To achieve the best performance,
>  ensure that all upcasters in the same chain \(where one's output is another's input\) work on the same content type.

If the content type conversion that you need is not provided by Axon you can always write one yourself using the `ContentTypeConverter` interface.

The `XStreamSerializer` supports dom4j as well as XOM as XML document representations. 
The `JacksonSerializer` supports Jackson's `JsonNode`.