# Event handling

Event listeners are the components that act on incoming events. They typically execute logic based on decisions that have been made by the command model. Usually, this involves updating view models or forwarding updates to other components, such as third party integrations. In some cases event handlers will throw events themselves based on \(patterns of\) events that they received, or even send commands to trigger further changes.
