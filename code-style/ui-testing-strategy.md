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

But we want all tests to support concurrent parallel invocation. Thus, tests will not depend on each other’s results, they won’t wait for each other, and their execution will be completed faster.

## Types of Tests

| Type    | Target | Run Against Prod | Real API | Mock API | Need Data Cleanup |
| -------- | ------- | ------- |------- |------- | ------- |
| E2E | Whole UI | Yes | Yes | No | Yes |
| Screenshot | UI Content | No | No | Yes | No |
| Components | Page | No | No | Yes (optional) | No |
|  | Container | No | No | Yes | No |
|  | Content | No | No | No | No |
| Unit | State, Utils | No | No | No | No |
| Contract Linting | API Contract Safety | No | No | No | No |

All Cypress test types are contained in .cy.ts files.

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
GIVEN user has completed the task and wants to track this in the time tracker
WHEN user adds a new task entry
AND user updates this task entry
THEN user should see the updated task entry in the time tracker
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

#### Example and location

E2E tests are in the cypress/e2e folder. The [Sick Leave Entry Happy Path](https://github.com/TourmalineCore/inner-circle-time-ui/blob/master/cypress/e2e/sick-leave-entries-happy-path.cy.ts) covers the second use case above.

### Screenshot

We aim at creating a stable robust UI and want keep it under control when the project is growing. We want to make sure that new components or pages don't affect the existing ones.
We also want to test manually as little as possible, including the stage of design conformance.

We writes these tests when we start to write HTML and add styles for a component.
Besides, in case you need to update the component, you need to regenerate its screenshot so it was up-to-date with the component's new appearance.

#### Threshold

We set the threshold to 0, because we want to detect any visual change to prevent unintended visual regressions.

#### What we test

- Components with static data. If a component has several states, we can test all of them with screenshots, e.g. an accordion in both collapsed and open state.
- The correct operation of the content component with the state.

We disable animations if they are present and disable autoplay for a video to make it static.
We test components on all regular breakpoints: mobile, tablet, tablet-xl, desktop, desktop-xl.

#### What we don't test

- Real dynamic data as it is too unpredictable
- Animations, videos
- Components' behavior
- Components between the breakpoints
- All browsers, only in Electron (we want to test it in different browsers in the future)
- Api requests

#### Example and location

Screenshot tests are located next to the component being tested. Their names also match the component names, but end with ScreenshotTest.

Each test has several settings:

- cy.wrap sets the screen parameters and the browser that opens the tests to correctly display the components being tested and eliminate device discrepancies
- wait for fonts to load before starting the test to avoid flags

If the components have permission-based logic, you need to create a mock wrapper, authService.AuthContext.Provider, with a mock token to correctly display the component being tested.

Example test in the books-ui service: [BookCopyScreenshotTest.cy.tsx](https://github.com/TourmalineCore/inner-circle-books-ui/blob/master/src/pages/book/BookCopyScreenshotTest.cy.tsx)

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

During development, when the API is not yet available and E2E tests fail, you can start testing network requests in container tests by mocking requests.

#### Content

We test how components behave depending on the state or the page, and how different components interact with each other. For example, showing a loader, disabling buttons, showing errors, filtering, sorting, etc.

#### Page components or shared components

We test the component's logic and its edge cases separately from the feature/page, for example, showing elements in a certain order.

#### What we don't test

- Logic that is configured outside the UI (nginx)
- Real interaction with the backend

#### Example and location

Tests are located next to the component being tested. Their files are named the same as the component's component files.

Examples:

- Page test: [AllBooksPage.cy.tsx from inner-circle-books-ui](https://github.com/TourmalineCore/inner-circle-books-ui/blob/master/src/pages/all-books/AllBooksPage.cy.tsx)

- Container tests:   
[EntryModalContainer.cy.tsx from inner-circle-time-ui](https://github.com/TourmalineCore/inner-circle-time-ui/blob/master/src/pages/time-tracker/sections/entry-modal/EntryModalContainer.cy.tsx) with validation and request dispatch checks  
[InvoicesContainer.cy.tsx from inner-circle-invoices-ui](https://github.com/TourmalineCore/inner-circle-invoices-ui/blob/master/src/pages/invoices/InvoicesContainer.cy.tsx) with checks for invoking requests with the correct parameters when interacting with the interface.
- Content test: [EntryModalContent.cy.tsx from inner-circle-time-ui](https://github.com/TourmalineCore/inner-circle-time-ui/blob/master/src/pages/time-tracker/sections/entry-modal/EntryModalContent.cy.tsx)
- Test for the component used on the page: [BookCard.cy.tsx from inner-circle-books-ui](https://github.com/TourmalineCore/inner-circle-books-ui/blob/master/src/pages/all-books/components/book-card/BookCard.cy.tsx)

### Unit

#### What we test

We test edge cases of the state's behavior and the logic of helper functions. For example, a function that formats a decimal number to two decimal places, the default state, going through all validation cases in the state.

#### What we don't test

- Components
- API request
- Logic that is configured outside the UI (nginx)

#### Example and location

Tests are located next to the component being tested. Their files are named the same as utils or state files.

[formatToTwoDecimalPlaces.cy.ts from inner-circle-time-ui](https://github.com/TourmalineCore/inner-circle-time-ui/blob/master/src/common/utils/format-to-two-decimal-places/formatToTwoDecimalPlaces.cy.ts)

[InvoicesState.cy.ts from inner-circle-invoices-ui](https://github.com/TourmalineCore/inner-circle-invoices-ui/blob/master/src/pages/invoices/state/InvoicesState.cy.ts)

#### Contract Linting

This is a static type of testing which allows to automatically check the contract between UI & Backend.

#### What we test

API contract beetwen UI & API.

#### What we don't test

Nothing except the API contract.

#### Example and location

Types and HTTP-client are generated from the Swagger in the API. After new API changes get into master, an npm package is published, and then a PR is opened in the UI repository. Lint runs in this PR, and if there are no errors, the new version of the npm package goes into the UI master. This lets us use a ready and correct contract in the UI and quickly spot contract changes in case of a breaking change.

Example: workflow [sync-js-client-with-api.yml](http://github.com/TourmalineCore/inner-circle-time-ui/blob/master/.github/workflows/sync-js-client-with-api.yml) from inner-circle-time-ui.

## How do we choose which tests to write?

1. Write a user story and list the happy path, business rules, edge cases, and errors.
2. Check existing coverage and identify which behavior needs a new test.
3. Choose the smallest test that can reproduce that behavior, using the test types above. 
4. Check that each test protects a separate responsibility. We don't need every test type for every change. Two tests can use the same input if they check different things, for example, a validation rule and its HTTP error response.

### Example: Return Book Flow

User story: the book has already been added to the library. The user checks it out and notes it in the system. After reading it, they return it and leave a review.

The API and take feature were ready.
The existing time-ui tests split the coverage as follows:

| Scenario | Where we test it | Why? |
| -------- | -------- | -------- |
| Takes a book, sees their name listed as a reader, returns it after reading, leaves a review, and sees the review on the book's page, but they are not listed as a reader | E2E: [Take and Return Book Flow](https://github.com/TourmalineCore/inner-circle-books-ui/blob/master/cypress/e2e/take-and-return-book-flow.cy.ts) | This is the core user flow. We've verified that taking and returning a book and adding a review work correctly|
| We check that the right content is shown, that the data (from state and props) is correct, and that the layout matches the design | Screenshot: [ReturnBookScreenshotTest](https://github.com/TourmalineCore/inner-circle-books-ui/blob/master/src/pages/return-book/ReturnBookScreenshotTest.cy.tsx) | We don't check all the content data in E2E, so we do it here. Also, this test is needed for TDD on the layout |
| We check the default state and edge cases, if there are any | Unit: [ReturnBookState](https://github.com/TourmalineCore/inner-circle-books-ui/blob/master/src/pages/return-book/state/ReturnBookState.cy.tsx) | These are edge cases of the state logic, testing all possible options |
| We check the disabled states depending on the state | Component content: [ReturnBookContent](https://github.com/TourmalineCore/inner-circle-books-ui/blob/master/src/pages/return-book/ReturnBookContent.cy.tsx) | At this layer we check how the components on the page work together. These checks are too much for E2E, and in screenshot tests we don't check different states. |

In unit tests for the state, it's better to test only edge cases. The main logic should be tested at higher levels (e2e, container, content). This is more reliable because there we check how the component works with the state. For example, if you wrote an initialize method in the state and tested it in unit tests, this doesn't guarantee that at the container level you used the data loading and passing this data to the state correctly. It also doesn't guarantee that you used the data from the state correctly in the content component.

### Example: fixing time saved with seconds

Bug: Tracked entries could be saved with seconds in their start or end time, although time must be stored with minute precision.

To fix this bug, we changed the logic of the `concatDateAndTimeToMinute` utils. We added logic to clear the seconds before sending data to the server. The regression test covers the following cases:

| Scenario | Where we test it | Why? |
| -------- | -------- | -------- |
| Before sending data to the server, the time and date chosen by the user are combined into the right format | Unit: [dateAndTime](https://github.com/TourmalineCore/inner-circle-time-ui/blob/master/src/common/utils/date-and-time/dateAndTime.cy.ts) | Because this function is the one that processes the data before sending it to the server |

TODO: Mark tests for bugs