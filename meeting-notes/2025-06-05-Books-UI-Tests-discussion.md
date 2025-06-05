# 2025-06-05 Books UI How Do We Test

## Question

Should we test Books UI with visual regression tests?

>Important: Visual regression tests aren't going to check the logic, behavior. This type of tests should be independent from network (API). It should be very limited number of such tests, agreed before development of a page.


## Context

- We have ready and pretty design. This is the first service in Inner Circle that will be developed like that. Before we styled as mockups and developed it differently for each service without a design before hand.

- Cypress is used everywhere for component and e2e tests. There are examples and best practices.

- Playwright was used at the first time on Pelican project. There were visual regression/pixel perfect tests with TDD. As well there were other types of tests on Playwright: API, E2E.

- Cypress didn't work very well with Next.js setup for visual regression testing. That is why we picked Playwright which didn't work well in component tests mode.


## Cypress vs Playwright

|    What    |             Cypress              |       Playwright       |
|------------|----------------------------------|------------------------|
|    E2E     |                +                 |           +            |
| Component  |                +                 | +- (more not than yes) |
|    a11y    |  +- (e.g. tabs cannot be tested) |           +            |
|   visual   | maybe (need to look for plugins) |           +            |
|  parallel  |  paid or manual ad-hoc solution  |           +            |
|    usage   |            everywhere            |      only Pelican      |


### Mad Idea

Cypress and Playwright in a single project!!!


### Risks of Introduction of Playwright

1. Component tests aren't really working with Next.js, might not also work with our Vite build setup.

2. Single service with Playwright and others with Cypress. Is it worth it?


### Benefits of Introduction of Playwright

1. We expect pixel perfect to work sort of out of the box.

2. We have prepared setup for Next.js including a11y testing.

3. Modern tool, more practical experience with it to chose wisely between Cypress and Playwright.


## Proposal

Try visual regression testing in Cypress to-dos-ui fork which will have the same Vite build setup.

Use case:

1. Global app styles: yellow background.

2. ToDosContent component css/scss with red font color.

3. Add some breakpoint changes for 768, 1024 for ToDosContent.

4. Try to make screenshots in Component mode.

We need to answer if Cypress works in component mode for our setup with visual regression testing.