# Reactor

Overlooking Axon Framework architecture you can notice that system using Axon Framework is Message Driven, Responsive, Resilient, Elastic.
According to [Reactive Manifesto](https://www.reactivemanifesto.org/) same is stated for Reactive Systems. 

Although we can that Axon Framework is type of reactive system, we can't say that Axon Framework is a reactive framework.

{% hint style="info" %}
Reactive programming is an approach to writing software that embraces asynchronous IO. Asynchronous I/O is a small idea that portends big changes for software. The idea is simple: alleviate inefficient resource utilization by reclaiming resources that would otherwise be idle as they waited for I/O activity. Asynchronous IO inverts the normal design IO processing: the clients are notified of new data instead of asking for it; this frees the client to do other things while waiting for new notifications. 
{% endhint %}

By nature, reactive API and Axon Framework are great fit as most of the Axon Framework Operations are async & non-blocking.
We chose to use Pivotal’s Reactor project and it's a good choice here, it builds on top of the Reactive Streams specification and its de-facto standard for Java enterprise & Spring applications.

With this extension we aim to make Axon a reactive framework by introduce pure Reactive API.


{% hint style="info" %}
Not all Axon Framework components offer reactive API. We are introducing Reactive API via this extension incrementally giving priority to components where users can benefit the most. 
{% endhint %}

To use the Reactor Extension, make sure that `axon-reactor` module is avalible on the classpath.

