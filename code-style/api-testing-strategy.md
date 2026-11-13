# Inner Circle API Testing Strategy

In principle we develop software according to Test-Driven Development (TDD). Inner Circle is not an exception from this rule.

We have a set of tests examples across various projects and services, we have experience working like that in frontend and backend development. However, we are still struggling to make the right decisions while testing our code. This strategy should facilitate the decision making, reduce the efforts, explain how to write only needed tests, speed up onboarding of the new team members by standardizing the approaches.

## Objectives

- List the available types of tests with refs and examples, explaining their areas of application and limitations.
- Answer when to write which types of tests in typical situations like implementing a new feature or fixing a bug.
- Minimize overlap of tests to reduce the efforts that we put into their writing and maintenance while keeping their high value. Tests are code, they come with a price, we need make an effort to make them maximally efficient yet reliable and automatically testable.
- Make it a piece of cake to write the testing and functional code in the current infrastructural and methodological setup.
- Tests must run stably. If a test is flaky, fixing it must be prioritized.

## Parallel Execution of Tests

All tests must support concurrent parallel invocation. Thus, tests don't rely on each other's results, they don't wait for each other as well and their execution completes faster.

## Types of Tests

| Type    | Target | Run Against Prod | Use Real DB | Need Data Cleanup | Language |
| -------- | ------- | ------- |------- |------- |------- |
| E2E | Whole API | Yes | Yes | Yes | Karate |
| Integrational with DB| Handler, Command, Query (everything that interacts with the Db) | No | Yes | No (Auto) | C# |
| Controller validation | Controller | No | No | No | C# |
| Integration with mocks | Handler | No | No | No | C# |
| Unit | Mapper, Validator, Calculator (everything that doesn’t interact with the Db) | No | No | No | C# |

### E2E

#### Why do we write these tests using Karate?

We want them to be stack-independent. Thus, you can write a backend service using any framework and language like .NET(C#), NestJS(JS), FastAPI(Python) or even C++ but the E2E tests language is the same.

Karate has a few more advantages:
- Declarative. Very close to UI-related use cases.
- Gherkin. Well-known language, human-readable, framework-agnostic.
- Support sophisticated scenarios using JavaScript or Java code snippets (when only declarative is not enough and a bit of imperative code snippets are needed).
- Can be used more intensively, e.g. for performance testing using the same set of tests (future possibility).

#### Unit of Parallelism

Karate scenario is a unit of parallelism of this type of tests. Scenarios are being executed in parallel. Thus, scenarios should be isolated from each other using different Tenants, Accounts, and more custom approaches. For instance, in time-api we can isolate different scenarios using different years, 2026 for one scenario and 2027 for another one.  

#### Production Execution

All E2E tests must support execution against Prod and Local.
The tests should use only test accounts for production tests.

However, there might be cases when some E2E tests cannot be executed against Local Env. For instance, that was the case with testing books-ui URL redirection logic that couldn't be tested locally because it was a DNS feature.

#### What we test

We test only the happy path - when everything goes according to plan and no unexpected situations happen, the core business flow. We limit the number of E2E tests to only the core business flow to avoid falling back into the situation when we test every change with a coarse-grained tests when it is an overkill and a lighter more fine-grained test is enough.

Basic rule: every endpoint must be called at least once in some E2E test. We try to write separate (atomic) realistic user paths scenarios in the system, instead of one big test that calls all the endpoints. This lets us immediately see where an error happened and makes tests faster through parallel execution.
 
Time to run the tests is also a limitation factor, yet minor at this phase. Tests against Prod are being executed faster than 5 mins, it seems ok.

Before writing an E2E test, we describe a use case, for example:

```
GIVEN I have planned a 14 days paid vacation.
WHEN I track my vacation in the time tracker.
THEN I realized that I need shift it for a month later.
```

```
GIVEN I was away for two hours.
WHEN I want a 30-minute make-up tomorrow and 1 hour 30 minutes the day after tomorrow.
THEN I realized I can't do the make-up tomorrow and want to move the 30-minute make-up to another day.
```

> **Note:** Describe use cases in business language. Do not use API or frontend terms, or mention buttons or endpoints.

Besides business cases, in E2E we also test that endpoints are protected by permissions, as well as CORS settings and that Swagger works.

#### What we don't test

Edge cases and negative scenarions.

#### Example and location

Karate tests are in the e2e folder, in .feature files. The [away with make-up time happy path](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/e2e/away-with-make-up-time-entries-happy-path.feature) covers the second use case above.

### Integrational

#### Why do we test with a real database?

We use a real database because it lets us test the code using the same database as in production. Unlike InMemoryDb, a real database lets us check real SQL queries, transactions, or business rules at the database level (constraints).

#### Test isolation

Each test runs in its own transaction with the Serializable isolation level, and the transaction is rolled back after successful or failed tests, so tests don't interfere with each other. This allows us not to clean up data at the beginning or end of tests.

TODO: finish after fixing the deadlock problems.

#### What we test

Logic of Commands, Queries, Handlers, and possibly other components that work with the database. Integration tests check everything that is not covered by E2E tests.

Commands and Queries — we test not the C# logic, but what happens in SQL: filtering, sorting, grouping, or business logic checked in the database (constraints).

Handlers — we test validation, work with external services, edge cases, and negative cases (for example, an external service returned null).

#### What we don't test

Integration tests don't duplicate the checks that are already in E2E.

#### Example and location

Database integration tests are next to the Commands, Queries, and Handlers in Application, in files ending with Tests.cs. They use the IntegrationTest attribute and inherit from IntegrationTestBase.

For example, [UpdateAwayWithMakeUpTimeEntryCommandTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Tracking/Handlers/UpdateAwayWithMakeUpTimeEntry/UpdateAwayWithMakeUpTimeEntryCommandTests.cs).

### Integrational with HTTP

#### Why do we use InMemoryDb in this type of test?

InMemoryDb is only a stub that allows the application to start. Normally, these tests do not access it.

For these tests we use InMemoryDb, because in these tests we don't need to save anything to the database. In this type of test, transaction logic is not implemented, so if we used a real database and tests were run before validation was added or if it was implemented incorrectly, data could be saved to the DB. To avoid adding cleanup logic or transactions, we decided to use InMemoryDb.

#### What we test

Contract validation at the controller level.

#### What we don't test

Business logic of the application or work with the database.

#### Example and location

HTTP integration tests are next to the controllers in Api/Features, in files ending with ValidationTests.cs. They use the IntegrationTest attribute and inherit from ControllerValidationTestsBase.

For example, [TrackingControllerValidationTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Api/Features/Tracking/TrackingControllerValidationTests.cs) checks HTTP 400 and the error response when the total make-up time does not match the absence duration.

### Integration tests with mocks

#### How do we test a handler without a database?

We move database logic to Commands and Queries and define interfaces for them. The handler receives these dependencies through its constructor. In tests, we replace them with mocks. This allows us to test the handler logic without a database. We test Commands and Queries separately with a real database.

#### What we test

Handler logic: validation, calls to dependencies, processing their results, edge cases, and negative cases.

#### What we don't test

SQL queries, database constraints, or work with real external services. We don't duplicate unit tests for calculators and mappers in the handler tests.

#### Example and location

Handler tests are next to the handler in Application/Features, in files ending with HandlerTests.cs. For example, [GetEmployeesTrackedTaskHoursHandlerTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Internal/Handlers/GetEmployeesTrackedTaskHours/GetEmployeesTrackedTaskHoursHandlerTests.cs) uses a mock of IGetTaskEntriesQuery.

> **Note:** This test currently uses the UnitTest attribute in time-api.

### Unit

#### What we test

Pure functions without interaction with the database (calculators, mappers, etc.). We fully test the logic of the function.

#### What we don't test
 
Everything that interacts with the database.

#### Example and location

Unit tests are next to the tested class, in files ending with Tests.cs, and use the UnitTest attribute. For example, see [MinutesConverterExtensionsTests in Core](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Core/Extensions/MinutesConverterExtensionsTests.cs).

## How do we choose which tests to write?

1. Write a user story and list the happy path, business rules, edge cases, and errors.
2. Check existing coverage and identify which behavior needs a new test.
3. Choose the smallest test that can reproduce that behavior, using the test types above. Use a real database only when database behavior matters, and HTTP when the API contract matters.
4. Check that each test protects a separate responsibility. We don't need every test type for every change. Two tests can use the same input if they check different things, for example, a validation rule and its HTTP error response.

### Example: a feature for planning make-up time

User story: As an employee, I want to split two hours of absence into a 30-minute make-up tomorrow and a 90-minute make-up the day after tomorrow. If my plans change, I want to move the 30-minute period to another day and keep the other period unchanged.

The existing time-api tests split the coverage as follows:

| Scenario | Where we test it | Why? |
| -------- | -------- | -------- |
| Plan two make-up periods, move one, and see the updated plan | E2E: [away with make-up time happy path](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/e2e/away-with-make-up-time-entries-happy-path.feature) | This is the core user flow We check that creation, update, and reading work together |
| Keep unchanged make-up entries and replace changed entries with new records | Integrational with DB: [UpdateAwayWithMakeUpTimeEntryCommandTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Tracking/Handlers/UpdateAwayWithMakeUpTimeEntry/UpdateAwayWithMakeUpTimeEntryCommandTests.cs) | The E2E flow checks the visible result This test checks which stored entries are preserved |
| The total make-up time matches or does not match the absence duration | Unit: [MakeUpTimeValidatorTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Validators/MakeUpTimeValidatorTests.cs) | These are input combinations for a pure validation rule They don't need HTTP or a database |
| The handler rejects a plan with a different total make-up time | Integrational with DB: [CreateAwayWithMakeUpTimeEntryHandlerTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Tracking/Handlers/CreateAwayWithMakeUpTimeEntry/CreateAwayWithMakeUpTimeEntryHandlerTests.cs) and [UpdateAwayWithMakeUpTimeEntryHandlerTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Tracking/Handlers/UpdateAwayWithMakeUpTimeEntry/UpdateAwayWithMakeUpTimeEntryHandlerTests.cs) | Checks that the handler applies the validator One representative invalid case per handler is enough; the validator tests cover the combinations |
| Creating a plan with mismatched total make-up time returns HTTP 400 with the expected error title and message | Integrational with HTTP: [TrackingControllerValidationTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Api/Features/Tracking/TrackingControllerValidationTests.cs) | Checks how the application error is returned to the caller |

The [create handler tests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Tracking/Handlers/CreateAwayWithMakeUpTimeEntry/CreateAwayWithMakeUpTimeEntryHandlerTests.cs) and [update handler tests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Tracking/Handlers/UpdateAwayWithMakeUpTimeEntry/UpdateAwayWithMakeUpTimeEntryHandlerTests.cs) currently use IntegrationTestBase and real Commands.

Access rules are covered by the separate [away with make-up time no permission tests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/e2e/away-with-make-up-time-entries-no-permissions-lead-to-unauthorized.feature).

### Example: fixing time saved with seconds

Bug: Tracked entries could be saved with seconds in their start or end time, although time must be stored with minute precision.

The fix adds the database constraint ck_entries_time_no_seconds and converts its violation to TimeNoSecondsException. A migration also removes seconds from existing entries. The regression test covers the following cases:

| Scenario | Where we test it | Why? |
| -------- | -------- | -------- |
| Reject a task entry with seconds in its start time or end time, with the expected exception and message | Integrational with DB: [CreateTaskEntryCommandTests](https://github.com/TourmalineCore/inner-circle-time-api/blob/master/Application/Features/Tracking/Handlers/CreateTaskEntry/CreateTaskEntryCommandTests.cs) | Checks that the real database rejects the entry through ck_entries_time_no_seconds and the command returns TimeNoSecondsException |

We choose a test with a real database because the fix depends on a SQL constraint. A unit test or a mock cannot check that constraint. We do not add another E2E test for these negative cases. This test does not cover the HTTP error response.

## Test data cleanup and stability

TODO: Describe test data cleanup after successful, failed, and interrupted runs, and isolation during parallel execution.
