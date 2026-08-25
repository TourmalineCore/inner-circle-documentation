# 009: Work with date in e2e

## Status
Proposed

## Context
We want our e2e tests (UI and API) to be able to run in parallel. For this, they must have non-overlapping dates so that they do not fail with a time overlap validation error.

Currently, we use a separate year for each test.

For example:

### Karate E2E tests 
| Test Name | Year |
| :--- | :--- |
| Task Entries - Happy Path | 2030 |
| Unwell Entries - Happy Path | 2029 |

Currently in the codebase Karate e2e tests go upward from 2029 to 2036.

### Cypress E2E tests 
| Test Name | Year |
| :--- | :--- |
| Task Entries - Happy Path | 2025 |
| Unwell Entries - Happy Path | 2024 |

Currently in the codebase Cypress e2e tests go down from 2029 to 2020.

The full list can be seen [here](/time-tracker/testing-strategy.md).

It is necessary to consider other options for the date selection strategy and understand whether it makes sense to change the current approach.

## Decision
After considering other alternatives, we decided not to change the current approach with a one-year step.

Pros:
- Simplicity of choosing a date for a test (we take a year that is not yet occupied without considering edge cases).
- The risk of error is minimal (within one year you can choose any date and even use several months without breaking anything).

Cons:
- Inconvenience during design and review (for example, if the year is far in the future, e.g. 2040, it is more difficult to find the days of the week that we want to check).
- In case we add the feature of the timesheet freeze that prevents tracking time for past months, our current tests will not pass this validations. We will have to rewrite all the tests, possbly making them dynamic, i.e. using relative dates (Date.today). However, replacing hardcoded dates with dynamic values is not a major rewrite.

## Alternatives

## A step of a month or several months
Instead of a unique year, use a combination of year + month, and also use different years for UI and API. In the case of API, start with 2025 and go up when months run out, and in UI start with 2024 and go down.

For example:

### Karate E2E tests 
| Test Name | Year |
| :--- | :--- |
| Task Entries - Happy Path | 2025-02 |
| Unwell Entries - Happy Path | 2025-01 |

### Cypress E2E tests 
| Test Name | Year |
| :--- | :--- |
| Task Entries - Happy Path | 2024-02 |
| Unwell Entries - Happy Path | 2024-01 |

Pros:
- Easier to find days of the week during design and review (at least while there are few tests).

Cons:
- Greater cognitive load when choosing dates (you need to consider not only the year but also the month).
- Some tests use several months at once (for example, UI e2e tests with the case when we move vacation to another month). This is more difficult to document (if several months are involved, they must be reserved and this must be reflected in the testing strategy).
- The risk of error is greater.

> Further reducing the step leads to even greater cognitive load and the chance of error becomes greater, and it is more difficult to document.

## Using different accounts or tenants
To ensure the tests parallelism, we can create several accounts instead of keeping the track of non-overlapping dates.

Pros:
- No need to manage date ranges or coordinate across tests.

Cons:
- Requires setting up and managing multiple test accounts/tenants.
- Harder to maintain.