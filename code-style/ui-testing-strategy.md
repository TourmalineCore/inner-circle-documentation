# Inner Circle UI Testing Strategy

In principle we develop software according to Test-Driven Development (TDD). Inner Circle is not an exception from this rule.

We have a set of tests examples across various projects and services, we have experience working like that in frontend and backend development. However, we are still struggling to make the right decisions while testing our code. This strategy should facilitate the decision making, reduce the efforts, explain how to write only needed tests, speed up onboarding of the new team members by standardizing the approaches.

## Objectives

- List the available types of tests with refs and examples, explaining their areas of application and limitations.
- Answer when to write which types of tests in typical situations like implementing a new feature or fixing a bug.
- Minimize overlap of tests to reduce the efforts that we put into their writing and maintenance while keeping their high value. Tests are code, they come with a price, we need make an effort to make them maximally efficient yet reliable and automatically testable.
- Make it a piece of cake to write the testing and functional code in the current infrastructural and methodological setup.
- Tests must run stably. If a test is flaky, fixing it must be prioritized.

## Parallel Execution of Tests

Cypress runs tests one by one by default. To run tests in parallel, you need extra configuration. At the moment, we don't have parallel test execution set up.

But we want all tests must support concurrent parallel invocation. Thus, tests will not depend on each other’s results, they won’t wait for each other, and their execution will be completed faster.

## Types of Tests

| Type    | Target | Run Against Prod | Use Real DB |Need Data Cleanup |
| -------- | ------- | ------- |------- |------- |
| E2E | Whole UI | Yes | Yes | Yes |
| Screenshot | Content | No | No | No |
| Components | Page, Container, Content | No | No | No |
| Unit | State, Utils | No | No | No |
| Types Linting | API Contract Safety | No | No | No |

### E2E

#### Production Execution

All E2E tests must support execution against Prod and Local.
The tests should use only test accounts for production tests. 

However, there might be cases when some E2E tests cannot be executed against Local Env. For instance, that was the case with testing books-ui URL redirection logic that couldn't be tested locally because it was a DNS feature.

#### What we test

We test only the happy path - when everything goes according to plan and no unexpected situations happen, the core business flow. We limit the number of E2E tests to only the core business flow to avoid falling back into the situation when we test every change with a coarse-grained tests when it is an overkill and a lighter more fine-grained test is enough.

We try to write separate (atomic) realistic user paths scenarios in the system, instead of one big test that calls all the endpoints. This lets us immediately see where an error happened and makes tests faster through parallel execution.

Before writing an E2E test, we describe a use case, for example:

```
GIVEN empty time tracker table
WHEN user adds a new task entry
AND user clicks on this task entry for update
THEN user should see the updated task entry in the time tracking table
```

```
GIVEN user has a sick leave from Monday to Friday
AND user wants to track this sick leave in the time tracker
WHEN user adds the sick leave period from Monday to Friday
AND open existing sick leave and extends it until next Monday after the doctor's visit
THEN user should see 8-day sick leave
```

Besides business cases, in E2E we also test that redirects works (for example, those configured at the nginx level).

#### What we don't test

Edge cases and negative scenarions.

### Screenshot

We aim at creating a stable robust UI and want keep it under control when the project is growing. We want to make sure that new components or pages don't affect the existing ones.
We also want to test manually as little as possible, including the stage of design conformance.

We writes these tests when we start to write HTML and add styles for a component.
Besides, in case you need to update the component, you need to regenerate its screenshot so it was up-to-date with the component's new appearance.

#### Threshold

We set the threshold to 0, because we want to detect any visual change to prevent unintended visual regressions.

#### What we test

Components with static data.
If a component has several states, we can test all of them with screenshots, e.g. an accordion in both collapsed and open state.

We disable animations if they are present and disable autoplay for a video to make it static.
We test components on all regular breakpoints: mobile, tablet, tablet-xl, desktop, desktop-xl.

#### What we don't test

- Real dynamic data as it is too unpredictable
- Animations, videos
- Components' behavior
- Components between the breakpoints
- All browsers, only in Electron
- Api requests

### Components

Component tests cover layers such as: page, container, content, or a regular component (for example, Button).

#### What we test

#### Page

We test high-level logic, for example, passing data from one state to another.

#### Container

We test the logic of working with requests and their responses.

For example:
- When the "Submit" button is clicked, a request is triggered
- The incoming data is saved to the state
- If an error response comes back, it is shown on the UI

#### Content

We test how components behave depending on the state or the page, and how different components interact with each other. For example, showing a loader, disabling buttons, showing errors, filtering, sorting, etc.

#### Page components or shared components

We test the component's logic and its edge cases separately from the feature/page, for example, showing elements in a certain order.

#### What we don't test

- Logic that is configured outside the UI (nginx)
- Real interaction with the backend

### Unit

#### What we test

We test edge cases of the state's behavior and the logic of helper functions. For example, a function that formats a decimal number to two decimal places, the default state, going through all validation cases in the state, and so on.

#### What we don't test

- Components
- API request
- Logic that is configured outside the UI (nginx)

#### Types Linting

This is a static type of testing which allows to automatically check the contract between UI & Backend.

Types are generated from the Swagger in the API. After new API changes get into master, an npm package is published, and then a PR is opened in the UI repository. Lint runs in this PR, and if there are no errors, the new version of the npm package goes into the UI master. This lets us use a ready and correct contract in the UI and quickly spot contract changes in case of a breaking change.

#### What we test

API contract beetwen UI & API.

#### What we don't test

Nothing except the API contract.