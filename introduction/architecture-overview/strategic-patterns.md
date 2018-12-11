# Strategic Patterns and Concepts

Domain Driven Design is a set of tools that assist us in designing and implementing software that delivers high value, both strategically and tactically. 

Strategic design is used like broad brushstrokes prior to getting into the details of implementation.

We will segregate our domain model by using the strategic design pattern called Bounded Contexts, and we will develop a Ubiquitous Language as our domain model within an explicitly Bounded Context.

Inside Bounded Context we use CQRS (and Evensourcing) architectural patterns to implement domain model. On a high level view we want to converge to Microservices architectural style, mapping multiple Bounded Contexts in one solution.

## Bounded Context

Rather than writing concrete definition of what Bounded Context really is, we prefer to follow particular set of rules:

 - Explicitly define the context within which a model applies.
 - Explicitly set boundaries in terms of team organization, usage within specific parts of the application, and physical manifestations such as code bases and database schemas.
 - Keep the model strictly consistent within these bounds, but don’t be distracted or confused by issues outside.

## Subdomains

Domain can be very big. For example, the whole `World` is a domain. Model for this domain is a concrete `World Map(Chart)`. Usually we want to split this domain to subdomains `Country Maps` so we can understand it better, and most probably we want to use a programming language to implement this model.

There are numerous techniques that can help us to decompose big domain. [Event Storming](https://www.eventstorming.com/book/) is a particularly interesting one. It is a workshop format for quickly exploring complex business domains.

## Context Mapping

Bounded contexts (and teams that produce them) can be in different relationships:

 - partnership (two contexts/teams succeed or fail together)
 - customer-supplier (two teams in upstream/downstream relationship - upstream can succeed interdependently of downstream team)
 - conformist (two teams in upstream/downstream relationship - upstream has no motivation to provide to downstream, and downstream team does not put effort in translation)
 - shared kernel (sharing a part of the model - must be kept small)
 - separate ways (cut them loose)
 - anticorruption layer

Context mapping is very important part of the process. For example, if consumer use the event types (e.g., classes) of an event publisher rather then depending only on the schema of the events (Published Language) then two contexts are most proabbably in a conformist relationship. If we are applying Microservices architectural style  teams should be more independent and we should try to escape from this relationship by publishing events as JSON, or perhaps a more economical object format.
