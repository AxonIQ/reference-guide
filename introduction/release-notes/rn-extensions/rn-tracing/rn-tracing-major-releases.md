# Major Releases

This page notes all enhancements and features that we have introduced to our major releases of the Axon Tracing Extension.

## Release 4.1 - Milestone

* The constructors of the `TracingCommandGateway` and `TracingQueryGateway` are now protected, since issue [#9](https://github.com/AxonFramework/extension-tracing/issues/9).
  This change allows users to extend these classes if necessary.

* As off issue [#7](https://github.com/AxonFramework/extension-tracing/issues/7), the spans now contain defaults operation names and tags.

You can find a complete list of the changes [here](https://github.com/AxonFramework/extension-tracing/issues?q=is%3Aclosed+milestone%3A%22Release+4.1%22)

Note that this extension currently is in a _milestone_ state.
As such, users should consider we might introduce API changes in future releases.

## Release 4.0 - Milestone

We introduced the Tracing extension with lots of help from our contributor Christophe Bouhier at Trifork.
The tracing logic used originates from the [Open Tracing API](https://opentracing.io/).

You can find a complete list of all changes [here](https://github.com/AxonFramework/extension-tracing/issues?q=is%3Aclosed+milestone%3A%22Release+4.0%22).

It's currently in a _milestone_ state, as it doesn't trace all `QueryGateway` operations.
As such, users should consider we might introduce API changes in future releases.
