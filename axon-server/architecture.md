# Architecture

## Overview
In a message-driven micro-services environment it is extremely important that communication
between services is efficient, reliable and easy to manage and monitor. Routing of messages
should not require any manual configuration and adding new services should be easy to do.

With Axon Framework the internals of communication are hidden for the application developers.
AxonServer provides this same experience in a distributed environment. It is an easy-to-use,
easy-to-manage platform to handle all events, commands and queries.

AxonServer has knowledge about the different types of messages that are being exchanged:
events, sent from one service to one or many other services, to notify the services that something has happened,
commands, sent to one service to do something, potentially waiting for a result
queries, sent to one or more services to retrieve information.

Each of these messages requires a different strategy, all supported by AxonServer.

Applications connect to the messaging platform and register their capabilities. One application may be able to execute a specific set of commands, another may handle a number of queries. There may be multiple instances of the same application connected to the messaging platform, each instance having the same capabilities.
