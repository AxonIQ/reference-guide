# Reactor

Overlooking Axon Frameworks architecture, you can notice that in general systems using the framework are Message Driven, Responsive, Resilient and Elastic.
According to [Reactive Manifesto](https://www.reactivemanifesto.org/), the same holds for Reactive Systems in general. 

Although we can state that Axon Framework is a _type_ of reactive system, we can't say that Axon Framework is fully reactive.

{% hint style="info" %}
Reactive programming is an approach to writing software that embraces asynchronous I/O. 
Asynchronous I/O is a small idea that portends big changes for software. 
The idea is simple: alleviate inefficient resource utilization by reclaiming resources that would otherwise be idle, as they waited for I/O activity. 
Asynchronous I/O inverts the normal design I/O processing: the clients are notified of new data instead of asking for it, which frees the client to do other work while waiting for these notifications. 
{% endhint %}

By nature, a reactive API and Axon Framework are a great fit, as most of framework's operations are async and non-blocking.
We chose to use Pivotal’s Reactor project to provide a reactive extension to Axon.
Reactor builds on top of the Reactive Streams specification and is the de-facto standard for Java enterprise & Spring applications.
As such, we feel it to be a great fit to provide an extension in.

With this extension we aim to make Axon reactive, by introduce a pure Reactive API.

{% hint style="info" %}
Not all Axon components offer a reactive API, yet. 
We are introducing Reactive API through this extension incrementally, giving priority to components where users can benefit the most. 
{% endhint %}

To use the Reactor Extension, make sure that `axon-reactor` module is available on the classpath.
