# Development Mode

You can start Axon Server in development mode which enables some features for development convenience.

These features can be enabled by configuring the following property:

```text
axoniq.axonserver.devmode.enabled=true
```

## Resetting Events

Whilst creating new features it can be convenient to restore Axon Server to a clean state with no events stored. This can also be helpful when writing and running integration tests against your system. Please note that data which is not stored in Axon Server \(e.g. tracking tokens\) is not deleted from this feature. These will have to be deleted or reset manually.

Resetting Axon Server can be done via the [CLI](../administration/command-line-interface.md) as well as the UI and REST interface.

> Note: This feature is disabled when Axon Server is running in cluster mode

