# Checklist before Deploying to Prod

Before you deploy to prod, make sure that:

## 1. Frontend
- all tests pass successfully, including screenshot tests if present
- necessary manual tests have been made on UI:
    - content overflow tests
    - visual testing in different browsers and breakpoints (including wide screens)
    - all functionality is working
    - buttons and links function as they should
    - a11y is checked by tabbing through the page and by a screen reader
- performance audit is made (e.g. using Lighthouse in Dev Tools)
- accessability audit is made (e.g. using Lighthouse in Dev Tools)
- CORS & CSP policies are followed
- if there is metrics included in the website, it is working
- privacy policy is present on the website
- sitemap is included
- robots.txt is included
- you have a list of environment variables for deploy

## 2. Backend
- all tests pass successfully
- all controllers are covered by auth
- you have a list of environment variables for deploy

## 3. DevOps
- you have a rollback plan in case newly deployed functionality is broken  and backups (if applicable)
- you have a step-by-step guideline of deploy
- default credentials are not used, instead prod secure credentials are used
- complex passwords are used for database