# Inner Circle API Testing Strategy

In principle we develop software according to Test-Driven Development (TDD). Inner Circle is not an exception from this rule.

We have a set of tests examples across various projects and services, we have experience working like that in frontend and backend development. However, we are still struggling to make the right decisions while testing our code. This strategy should facilitate the decision making, reduce the efforts, explain how to write only needed tests, speed up onboarding of the new team members by standardizing the approaches.

## Objectives

- List the available types of tests with refs and examples, explaining their areas of application and limitations.
- Answer when to write which types of tests in typical situations like implementing a new feature or fixing a bug.
- Minimize overlap of tests to reduce the efforts that we put into their writing and maintenance while keeping their high value. Tests are code, they come with a price, we need make an effort to make them maximally efficient yet reliable and automatically testable.
- Make it a piece of cake to write the testing and functional code in the current infrastructural and methodological setup.

## Parallel Execution of Tests

All tests must support concurrent parallel invocation. Thus, tests don't rely on each other's results, they don't wait for each other as well and their execution completes faster.

## Types of Tests

| Type    | Target | Run Against Prod | Use Real DB | Need Data Cleanup | Language |
| -------- | ------- | ------- |------- |------- |------- |
| E2E  | Whole API    | Yes    | Yes    | Yes    | Karate |
| Integrational | Handler, Command, Query (everything that interacts with the Db) | No    | Yes    | No (Auto) | C# |
| Integrational with HTTP | Controller | No    | No    | No    | C# |
| Unit | Mapper, Validator, Calculator (everything that doesn’t interact with the Db) | No    | No    | No    |  C# |


TODO: Rules of a good test.

### E2E

#### Why do we write these tests using Karate?

We want them to be stack-independent. Thus, you can write a backend service using any framework and language like .NET(C#), NestJS(JS), FastAPI(Python) or even C++ but the E2E tests language is the same.

Karate has a few more advantages:
- Declarative. Very close to UI-related use cases.
- Gherkin. Well-known language, human-readable, framework-agnostic.
- Support sophisticated scenarios using JavaScript or Java code snippets (when only declarative is not enough and a bit of imperative code snippets are needed).
- Can be used more intensively, e.g. for performance testing using the same set of tests (future possibility).

#### Unit of Parallelism

Karate scenario is a unit of parallelism of this type of tests. Scenarios are being executed in parallel. Thus, scenarios should be isolated from each other using different Tenants, Accounts, and more custom approaches. For instance, in time-api we can isolate different scenarios using different week days, Monday for one scenario and Tuesday for another one.  

#### Production Execution

All E2E tests must support execution against Prod and Local. 

However, there might be cases when some E2E tests cannot be executed against Local Env. For instance, that was the case with testing books-ui URL redirection logic that couldn't be tested locally because it was a DNS feature.

#### What we test

We test only the happy path - when everything goes according to plan and no unexpected situations happen, the core business flow. We limit the number of E2E tests to only the core business flow to avoid falling back into the situation when we test every change with a coarse-grained tests when it is an overkill and a lighter more fine-grained test is enough. Basic rule: every endpoint must be called at least once in some E2E test. We try to write separate (atomic) realistic user paths scenarios in the system, instead of one big test that calls all the endpoints. This lets us immediately see where an error happened and makes tests faster through parallel execution. Time to run the tests is also a limitation factor, yet minor at this phase. Tests against Prod are being executed faster than 5 mins, it seems ok.

Before writing an E2E test, we describe a use case, for example:

```
I have planned a 14 days paid vacation.
I track my vacation in the time tracker.
Then, I realized that I need shift it for a month later.
```

```
I was away for two hours.
I want a 30-minute make-up tomorrow and 1 hour 30 minutes the day after tomorrow.
Then I realized I can't do the make-up tomorrow and want to move the 30-minute make-up to another day.
```

Besides business cases, in E2E we also test that endpoints are protected by permissions, as well as CORS settings and that Swagger works.

#### What we don't test

Edge cases with different deviations of input data.

### Integrational

#### Why do we test with a real database?

We use a real database because it lets us test the code using the same database as in production.
Unlike InMemoryDb, a real database lets us check real SQL queries, transactions, or business rules at the database level (constraints).

#### Test isolation

Each test runs in its own transaction with the Serializable isolation level, and after the test passes, the transaction is rolled back, so tests don't interfere with each other. This allows us not to clean up data at the beginning or end of tests.

TODO: finish after fixing the deadlock problems.

#### What we test

We test the logic of Commands, Queries, Handlers, and possibly other components that work with the database. Integration tests check everything that is not covered by E2E tests.

Commands and Queries — we test not the C# logic, but what happens in SQL: filtering, sorting, grouping, or business logic checked in the database (constraints).

Handlers — we test validation, work with external services, edge cases, and negative cases (for example, an external service returned null).

#### What we don't test

Integration tests don't duplicate the checks that are already in E2E.

### Integrational with HTTP

#### Why do we use InMemoryDb in this type of test?

For these tests we use InMemoryDb, because in these tests we don't need to save anything to the database. In this type of test, transaction logic is not implemented, so if we used a real database and tests were run before validation was added or if it was implemented incorrectly, data could be saved to the DB. To avoid adding cleanup logic or transactions, we decided to use InMemoryDb.

#### What we test

We test contract validation at the controller level.

#### What we don't test

We don't test the business logic of the application or work with the database.

### Unit
