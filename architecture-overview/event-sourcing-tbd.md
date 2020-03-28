# Event Sourcing \(TBD\)

In a traditional way of storing an application’s state, we capture the current state and store it in some relational or [NoSQL](https://en.wikipedia.org/wiki/NoSQL) database. While this approach is really straightforward, it does not provide a way of deducting how we got to that current state. Of course, one may argue that we can have a separate model for keeping the history of actions that lead to the current state, but besides the additional complexity, these two models could easily go different paths and start being inconsistent \(which is basically an [update anomaly](https://www.sqa.org.uk/e-learning/MDBS01CD/page_22.htm)\).

Event Sourcing is a way of storing an application’s state through the history of events that have happened in the past. The current state is reconstructed based on the full history of events, where each event represents a change or fact in our application. Events give us a single source of truth about what happened in our application. It is especially beneficial in applications that need to provide a full audit log to external reviewers. The current state is called _Materialized state_ in some resources \(see _Figure 1_\).  


