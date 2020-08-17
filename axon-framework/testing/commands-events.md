# Commands / Events

One of the benefits of CQRS, and especially that of event sourcing, is that it is possible to express tests purely in terms of events and commands. Both being functional components, events and commands have clear meaning to the domain expert or business owner. Not only does this mean that tests expressed in terms of events and commands have a clear functional meaning, it also means that they hardly depend on any implementation choices.

The features described in this chapter require the `axon-test` module, which can be obtained by configuring a maven dependency \(use `<artifactId>axon-test</artifactId>` and `<scope>test</scope>`\) or from the full package download.

The fixtures described in this chapter work with any testing framework, such as JUnit and TestNG.

## Command Model Testing

The command handling component is typically the component in any CQRS based architecture that contains the most complexity. Being more complex than the others, this also means that there are extra test related requirements for this component.

Although being more complex, the API of a command handling component is fairly easy. It has a command coming in, and events going out. In some cases, there might be a query as part of command execution. Other than that, commands and events are the only part of the API. This means that it is possible to completely define a test scenario in terms of events and commands. Typically, in the shape of:

* Given certain events in the past,
* When executing this command,
* Expect these events to be published and/or stored

Axon Framework provides a test fixture that allows you to do exactly that. The `AggregateTestFixture` allows you to configure a certain infrastructure, composed of the necessary command handler and repository, and express your scenario in terms of "given-when-then" events and commands.

> **Focus of a Test Fixture**
>
> Since the unit of testing here is the aggregate, `AggregateTestFixture` is meant to test one aggregate only. So, all commands in the `when` \(or `given`\) clause are meant to target the aggregate under test fixture. Also, all `given` and `expected` events are meant to be triggered from the aggregate under test fixture.

The following example shows the usage of the "given-when-then" test fixture with JUnit 4 on the `GiftCard` aggregate \(as defined [earlier](../axon-framework-commands/modeling/aggregate.md#basic-aggregate-structure)\):

```java
import org.axonframework.test.aggregate.AggregateTestFixture;
import org.axonframework.test.aggregate.FixtureConfiguration;

public class GiftCardTest {

    private FixtureConfiguration<GiftCard> fixture;

    @Before
    public void setUp() {
        fixture = new AggregateTestFixture<>(GiftCard.class);
    }

    @Test
    public void testRedeemCardCommand() {
        fixture.given(new CardIssuedEvent("cardId", 100))
               .when(new RedeemCardCommand("cardId", "transactionId", 20))
               .expectSuccessfulHandlerExecution()
               .expectEvents(new CardRedeemedEvent("cardId", "transactionId", 20));
        /*
        These four lines define the actual scenario and its expected result. 
        The first line defines the events that happened in the past. 
        These events define the state of the aggregate under test.
        In practical terms, these are the events that the event store returns
         when an aggregate is loaded. 

        The second line defines the command that we wish to execute against our system. 

        Finally, we have two more methods that define expected behavior. 
        In the example, we use the recommended void return type. 
        The last method defines that we expect a single event as result
         of the command execution.
        */
    }
}
```

The "given-when-then" test fixture defines three stages: configuration, execution and validation. Each of these stages is represented by a different interface: `FixtureConfiguration`, `TestExecutor` and `ResultValidator`, respectively.

> **Fluent Interface**
>
> To make optimal use of the migration between these stages, it is best to use the fluent interface provided by these methods, as shown in the example above.

### Test Setup

During the configuration phase \(i.e. before the first "given" is provided\), you provide the building blocks required to execute the test. Specialized versions of the event bus, command bus and event store are provided as part of the fixture by default. There are accessor methods in place to obtain references to them. Any command handlers not registered directly on the aggregate need to be explicitly configured using the `registerAnnotatedCommandHandler` method. Besides an Annotated Command Handler, you can register a wide variety of components and settings that define how the infrastructure around the Aggregate's test should be set up, consisting out of the following:

* `registerRepository`:

  Registers a custom Aggregate [`Repository`](commands-events.md).

* `registerRepositoryProvider`:

  Registers a `RepositoryProvider` used to [spawn new Aggregates](../axon-framework-commands/modeling/aggregate-creation-from-another-aggregate.md).

* `registerAggregateFactory`:

  Registers a custom [`AggregateFactory`](commands-events.md).

* `registerAnnotatedCommandHandler`:

  Registers a Annotated Command Handler object.

* `registerCommandHandler`:

  Registers a `MessageHandler` of `CommandMessage`.

* `registerInjectableResource`:

  Registers a resource which can be injected in to message handling members.

* `registerParameterResolverFactory`:

  Registers a [`ParameterResolverFactory`](commands-events.md) to the test fixture.

  This method is used to complement the default `ParameterResolvers` with custom `ParameterResolver`.

* `registerCommandDispatchInterceptor`:

  Registers a command [`MessageDispatchInterceptor`](commands-events.md).

* `registerCommandHandlerInterceptor`:

  Registers a command [`MessageHandlerInterceptor`](commands-events.md).

* `registerDeadlineDispatchInterceptor`:

  Registers a [`DeadlineMessage`](commands-events.md) `MessageDispatchInterceptor`.

* `registerDeadlineHandlerInterceptor`:

  Registers a [`DeadlineMessage`](commands-events.md) `MessageHandlerInterceptor`.

* `registerFieldFilter`:

  Registers a `Field` filter used when comparing objects in the "then" phase.

* `registerIgnoredField`:

  Registers a field that should be ignored for a given class when state equality is performed.

* `registerHandlerDefinition`:

  Registers a custom [`HandlerDefinition`](../../appendices/message-handler-tuning/handler-enhancers.md) to the test fixture.

* `registerHandlerEnhancerDefinition`:

  Registers a custom [`HandlerEnhancerDefinition`](../../appendices/message-handler-tuning/handler-enhancers.md) to the test fixture.

  This method is used to complement the default `HandlerEnhancerDefinition` with a custom `HandlerEnhancerDefinition`.

* `registerCommandTargetResolver`:

  Registers a `CommandTargetResolver` to the test fixture.

Once the fixture is configured, you can define the "given" events. The test fixture will wrap these events as `DomainEventMessage`s. If the "given" event implements `Message`, the payload and meta data of that message will be included in the `DomainEventMessage`, otherwise the given event is used as payload. The sequence numbers of the `DomainEventMessage` are sequential, starting at `0`. If no prior activity is expected, the `givenNoPriorActivity()` can be used as the starting point.

Alternatively, you may also provide commands as "given" scenario. In that case, the events generated by those commands will be used to event source the aggregate when executing the actual command under test. Use the "`givenCommands(...)`" method to provide command objects.

A last option for the "given" phase, is providing the state of an Aggregate directly. This is not recommended in case of Event Sourcing, and only in cases where reconstructing aggregates based on Commands or Events is unfeasible or when a [State-Stored Aggregate](../axon-framework-commands/modeling/state-stored-aggregates.md) is used. Use `fixture.givenState(() -> new GiftCard())` to define the initial state.

### Test Execution Phase

The execution phase allows you two entry points towards the [validation phase](commands-events.md#validation-phase). Firstly, you can provide a Command to be executed against the command handling component. Similar as with the given Events, if the provided Command is of type `CommandMessage` it will be dispatched as is. The behavior of the invoked handler \(either on the aggregate or as an external handler\) is monitored and compared to the expectations registered in the validation phase.

Secondly, it is possible to elapse a certain time span with the `whenThenTimeElapses(Duration)` and `whenThenTimeAdvancesTo(Instant)` handles. These support testing the publication of `DeadlineMessages`, as is further defined in [this](../deadlines/) chapter.

Note that only activities that occur during the test _execution_ phase are monitored. Any Events or side-effects generated during the "given" phase are not considered in the validation phase.

> **Illegal State Change Detection**
>
> During the execution of the test, Axon attempts to detect any illegal state changes in the aggregate under test. It does so by comparing the state of the aggregate after the command execution to the state of the aggregate if it sourced from all "given" and stored events. If that state is not identical, this means that a state change has occurred outside of an aggregate's event handler method. Static and transient fields are ignored in the comparison, as they typically contain references to resources.
>
> You can switch detection in the configuration of the fixture with the `setReportIllegalStateChange()` method.

### Validation Phase

The last phase is the validation phase, which allows you to check on the activities of the command handling component. This is generally done purely in terms of return values and events.

#### Validating Command Result

The test fixture allows you to validate return values of your command handlers. You can explicitly define the expected return value, or simply require that the method successfully returned. You may also express any exceptions you expect the CommandHandler to throw.

The following methods are available for validating Command Results:

* `fixture.expectSuccessfulHandlerExecution()`:

  Validates that the handler returned a regular response, which was not marked as an exceptional response.

  The exact response is not evaluated.

* `fixture.expectResultMessagePayload(Object)`:

  Validates that the handler returned a successful response, with a payload equal to the given payload.

* `fixture.expectResultMessagePayloadMatching(Matcher)`:

  Validates that the handler returned a successful response, with a payload matching the given Matcher

* `fixture.expectResultMessage(CommandResultMessage)`:

  Validates that the `CommandResultMessage` received has equal payload and meta data to that of given message.

* `fixture.expectResultMessageMatching(Matcher)`:

  Validates that the `CommandResultMessage` matches the given Matcher.

* `fixture.expectException(Matcher)`:

  Validates that the command handling result is an exceptional result, and that the exception matches the given `Matcher`.

* `fixture.expectException(Class)`:

  Validates that the command handling result is an exceptional result with the given type of exception.

* `fixture.expectExceptionMessage(String)`:

  Validates that the command handling result is an exceptional result and the exception message is equal to the given message.

* `fixture.expectExceptionMessage(Matcher)`:

  Validates that the command handling result is an exceptional result and the exception message matches the given Matcher.

#### Validating Published Events

The other component is validation of published events. There are two ways of matching expected events.

The first is to pass in event instances that need to be literally compared with the actual events. All properties of the expected events are compared \(using `equals()`\) with their counterparts in the actual Events. If one of the properties is not equal, the test fails and an extensive error report is generated.

The other way of expressing expectancies is using "Matchers" \(provided by the Hamcrest library\). `Matcher` is an interface prescribing two methods: `matches(Object)` and `describeTo(Description)`. The first returns a boolean to indicate whether the matcher matches or not. The second allows you to express your expectation. For example, a "GreaterThanTwoMatcher" could append "any event with value greater than two" to the description. Descriptions allow expressive error messages to be created about why a test case fails.

Creating matchers for a list of events can be tedious and error-prone work. To simplify things, Axon provides a set of matchers that allow you to provide a set of event specific matchers and tell Axon how they should match against the list. These matchers are statically available through the abstract `Matchers` utility class.

Below is an overview of the available event list matchers and their purpose:

* **List with all of**: `Matchers.listWithAllOf(event matchers...)`

  This matcher will succeed if all of the provided event matchers match against at least one event in the list of actual events.

  It does not matter whether multiple matchers match against the same event,

  nor if an event in the list does not match against any of the matchers.

* **List with any of**: `Matchers.listWithAnyOf(event matchers...)`

  This matcher will succeed if one or more of the provided event matchers matches against one or more

  of the events in the actual list of events.

  Some matchers may not even match at all, while another matches against multiple others.

* **Sequence of Events**: `Matchers.sequenceOf(event matchers...)` Use this matcher to verify that the actual events are matched in the same order as the provided event matchers. It will succeed if each matcher matches against an event that comes after the event that the previous matcher matched against. This means that "gaps" with unmatched events may appear.

  If, after evaluating the events, more matchers are available, they are all matched against "`null`". It is up to the event matchers to decide whether they accept that or not.

* **Exact sequence of Events**: `Matchers.exactSequenceOf(event matchers...)`

  Variation of the "Sequence of Events" matcher where gaps of unmatched events are not allowed.

  This means each matcher must match against the event directly following the event the previous matcher matched against.

For convenience, a few commonly required event matchers are provided. They match against a single event instance:

* **Equal event**: `Matchers.equalTo(instance...)`

  Verifies that the given object is semantically equal to the given event.

  This matcher will compare all values in the fields of both actual and expected objects using a null-safe equals method.

  This means that events can be compared, even if they do not implement the equals method.

  The objects stored in fields of the given parameter _are_ compared using equals,

  requiring them to implement one correctly.

* **No more events**: `Matchers.andNoMore()` or `Matchers.nothing()`

  Only matches against a `null` value.

  This matcher can be added as last matcher to the _exact_ sequence of events matchers to ensure that no unmatched events remain.

* **Predicate Matching**: `Matchers.matches(Predicate)` or `Matchers.predicate(Predicate)`

  Creates a Matcher that matches with values defined by the specified `Predicate`.

  Can be used in case the `Predicate` API provides a better means to validating the outcome.

Since the matchers are passed a list of event messages, you sometimes only want to verify the payload of the message. There are matchers to help you out:

* **Payload matching**: `Matchers.messageWithPayload(payload matcher)`

  Verifies that the payload of a message matches the given payload matcher.

* **Payloads matching**: `Matchers.payloadsMatching(list matcher)`

  Verifies that the payloads of the messages matches the given matcher.

  The given matcher must match against a list containing each of the messages payload.

  The payloads matching matcher is typically used as the outer matcher to prevent repetition of payload matchers.

Below is a small code sample displaying the usage of these matchers. In this example, we expect two events to be published. The first event must be a "ThirdEvent", and the second "aFourthEventWithSomeSpecialThings". There may be no third event, as that will fail against the "andNoMore" matcher.

```java
import org.axonframework.test.aggregate.FixtureConfiguration;

import static org.axonframework.test.matchers.Matchers.andNoMore;
import static org.axonframework.test.matchers.Matchers.equalTo;
import static org.axonframework.test.matchers.Matchers.exactSequenceOf;
import static org.axonframework.test.matchers.Matchers.messageWithPayload;
import static org.axonframework.test.matchers.Matchers.payloadsMatching;

class MyCommandModelTest {

    private FixtureConfiguration<MyCommandModel> fixture;

    public void testWithMatchers() {
        fixture.given(new FirstEvent(), new SecondEvent())
               .when(new DoSomethingCommand("aggregateId"))
               .expectEventsMatching(exactSequenceOf(
                   // we can match against the payload only:
                   messageWithPayload(equalTo(new ThirdEvent())),
                   // this will match against a Message
                   aFourthEventWithSomeSpecialThings(),
                   // this will ensure that there are no more events
                   andNoMore()
               ));

               // or if we prefer to match on payloads only:
               .expectEventsMatching(payloadsMatching(
                   exactSequenceOf(
                       // we only have payloads, so we can equalTo directly
                       equalTo(new ThirdEvent()),
                       // now, this matcher matches against the payload too
                       aFourthEventWithSomeSpecialThings(),
                       // this still requires that there is no more events
                       andNoMore()
                   )
               ));
   }
}
```

#### Validating Aggregate State

In certain circumstances, it may be desirable to validate the state in which an Aggregate was left after a test. This is especially the case in given-when-then scenario's where the _given_ represents an initial state as well, as is regular when using a [State-Stored Aggregate](../axon-framework-commands/modeling/state-stored-aggregates.md).

The fixture provides a method that allows verification of the state of the aggregate, as it is left after the [Execution Phase](commands-events.md#test-execution-phase) \(e.g. the _when_ state\), to be validated.

```java
fixture.givenState(() -> new GiftCard())
       .when(new RedeemCardCommand())
       .expectState(state -> {
           // perform assertions
       });
```

The `expectState` method takes a consumer of the Aggregate type. Use regular assertions provided by your test framework to assert the state of the given Aggregate. Any \(Runtime\) Exception or Error will fail the test case accordingly.

> **Event-Sourced Aggregate State Validation**
>
> State validation for testing Event Sourced Aggregates is considered bad practice. Ideally, the state of an Aggregate is completely opaque to the testing code, as only the behavior should be validated. Generally, the desire to validate state is an indication that a certain test scenario is missing from the test suite.

#### Validating Deadlines

The validation phase also provides the option to verify scheduled and met [deadlines](../deadlines/) for a given Aggregate instance. You can expect scheduled deadlines both through a `Duration` or an `Instant`, using explicit equals, a `Matcher` or just a deadline type to verify the deadline message.  
The following methods are available for validating Deadlines:

* `expectScheduledDeadline(Duration, Object)`:

  Explicitly expect a given `deadline` to be scheduled after the specified `Duration`.

* `expectScheduledDeadlineMatching(Duration, Matcher)`:

  Expect a deadline matching the `Matcher` to be scheduled after the specified `Duration`.

* `expectScheduledDeadlineOfType(Duration, Class)`:

  Expect a deadline matching the given type to be scheduled after the specified `Duration`.

* `expectScheduledDeadlineWithName(Duration, String)`:

  Expect a deadline matching the given deadline name to be scheduled after the specified `Duration`.

* `expectScheduledDeadline(Instant, Object)`:

  Explicitly expect a given `deadline` to be scheduled at the specified `Instant`.

* `expectScheduledDeadlineMatching(Instant, Matcher)`:

  Expect a deadline matching the `Matcher` to be scheduled at the specified `Instant`.

* `expectScheduledDeadlineOfType(Instant, Class)`:

  Expect a deadline matching the given type to be scheduled at the specified `Instant`.

* `expectScheduledDeadlineWithName(Instant, String)`:

  Expect a deadline matching the given deadline name to be scheduled at the specified `Instant`.

* `expectNoScheduledDeadlines()`:

  Expect that no deadlines are scheduled at all.

* `expectNoScheduledDeadlineMatching(Matcher)`:

  Expect no deadline matching the `Matcher` to be scheduled.

* `expectNoScheduledDeadlineMatching(Duration, Matcher)`:

  Expect no deadline matching the `Matcher` to be scheduled after the specified `Duration`.

* `expectNoScheduledDeadline(Duration, Object)`

  Explicitly expect no given `deadline` to be scheduled after the specified `Duration`.\`

* `expectNoScheduledDeadlineOfType(Duration, Class)`

  Expect no deadline matching the given type to be scheduled after the specified `Duration`.\`

* `expectNoScheduledDeadlineWithName(Duration, String)`

  Expect no deadline matching the given deadline name to be scheduled after the specified `Duration`.\`

* `expectNoScheduledDeadlineMatching(Instant, Matcher)`:

  Expect no deadline matching the `Matcher` to be scheduled at the specified `Instant`.

* `expectNoScheduledDeadline(Instant, Object)`

  Explicitly expect no given `deadline` to be scheduled at the specified `Instant`.\`

* `expectNoScheduledDeadlineOfType(Instant, Class)`

  Expect no deadline matching the given type to be scheduled at the specified `Instant`.\`

* `expectNoScheduledDeadlineWithName(Instant, String)`

  Expect no deadline matching the given deadline name to be scheduled at the specified `Instant`.\`

* `expectDeadlinesMet(Object...)`:

  Explicitly expect a `deadline` or several deadlines to have been met.

* `expectDeadlinesMetMatching(Matcher<List<DeadlineMessage>>)`:

  Expect a matching deadline or several matching deadlines to have been met.

