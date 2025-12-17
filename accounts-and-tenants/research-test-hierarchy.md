# Research test hierarchy

## Tests list with indication of where they are launched

| Service Name                   |    Prod   | Local-env |   Docker  |
| :----------------------------- | :-------: | :-------: | :-------: |
| accounts-api                   |     +     |     +     |     -     |
| auth-api                       |     -     |     +     |     +     |
| auth-ui                        |     +     |     +     |     -     |
| inner-circle-compensations-api |     +     |     +     |     +     |
| inner-circle-compensations-ui  |     +     |     +     |     -     |
| inner-circle-books-api         |     +     |     +     |     -     |
| inner-circle-books-ui          |     +     |     +     |     -     |
| inner-circle-documents-api     |     -     |     +     |     +     |
| inner-circle-layout-ui         |     -     |     +     |     -     |
| inner-circle-local-env         |     -     |     +     |     -     |


## Tests description

### accounts-api

**usings**
- karate e2e test via local-env
- prod

**name:** E2E-basic-flow.feature

**prerequisite:** account with permissions
1. ViewAccounts
2. ManageAccounts
3. IsAccountsHardDeleteAllowed
4. ViewRoles
5. ManageRoles
6. CanManageTenants
7. IsTenantsHardDeleteAllowed

**steps:**
- authentication
- create tenant
- create role
- create account
- verify account
- delete account by corporateEmail created in this test
- delete role by roleId from this test
- delete tenant by tenantId from this test

**result:** shouldn't affect other tests


### auth-api

**usings**
- karate e2e test via local-env
- karate e2e test via docker-compose

**name:** check-employeeId-in-token.feature

**prerequisite:** active account with at least ViewPersonalProfile permission for example

**steps:**
- authentication
- check auth token after authentication

**result:** shouldn't affect other tests


### auth-ui

**usings**
- cypress e2e test via local-env
- prod

**prerequisite:** active account with at least ViewPersonalProfile permission for example

**name:** smoke.cy.ts

**steps:**
- check auth validation (you can't login with incorrect credentials)
- redirect from auth page after successful auth
- logout

**result:** shouldn't affect other tests


### inner-circle-compensations-api

**usings**

- karate e2e test via local-env
- karate e2e test via docker-compose
- prod

**prerequisite:** account with permissions
1. CanRequestCompensations
2. CanManageCompensations
3. IsCompensationsHardDeleteAllowed

**name:** E2E-Test-Flow.feature

**steps:**
- authentication
- get all personal compensations
- get types
- create a wrong compensation
- delete the wrong compensation by id from this test
- get all personal compensations
- create compensation
- get all personal compensations to check compensation from previous step
- get all employees compensations
- update compensation status by id from this test
- get all personal compensations and check status of compensation from this test
- TODO: add cleanup after test

**result:** shouldn't affect other tests


### inner-circle-compensations-ui

**usings**

- cypress e2e test via local-env
- prod

**prerequisite:** account with permissions
1. CanRequestCompensations
2. CanManageCompensations
3. IsCompensationsHardDeleteAllowed

**name:** compensations-smoke.cy.ts

**steps:**
- authentication
- remove compensations where comment starts with `[E2E-SMOKE]`
- add constant newCompensationComment = `[E2E-SMOKE] ${new Date()}`
- check that personal table doesn't contain newCompensationComment
- create compensation
- check that personal table contains compensation with newCompensationComment and "unpaid" status
- find compensation in all compensations table by newCompensationComment
- mark this compensation as paid
- check that personal table contains compensation with newCompensationComment and "paid" status
- remove compensations where comment starts with `[E2E-SMOKE]`

**result:** shouldn't affect other tests


### inner-circle-books-api

**usings**
- karate e2e test via local-env
- prod

**prerequisite:** account with permissions
1. CanViewBooks
2. CanManageBooks
3. IsBooksHardDeleteAllowed

**1. name:** CRUD-operations-E2E.feature

**steps:**
- authentication
- create book with randomName = 'Test-book-' + Math.random()
- check by id from first step that the book is created and there are 2 book copies
- edit by id from first step the book's details with editedName = 'Test-edited-book' + Math.random()
- get the edited book by id from first step and verify the details have changed
- delete the book by id from first step (soft delete)
- verify the book with id from first step no longer in the list
- delete the book by id from first step (hard delete)

**result:** shouldn't affect other tests

**2. name:** take-book-flow-e2e.feature

**steps:**
- authentication
- create book with randomName = 'Test-book-' + Math.random()
- get book and copies by bookId from first step
- get book copy by bookCopyId from second step
- cleanup: delete the book by id from first step (hard delete)

**result:** shouldn't affect other tests

**3. name:** book-copy-happy-path.feature

**steps:**
- authentication
- create book with randomName = 'Test-book-' + Math.random()
- get book and copies by bookId from first step
- get book copy by bookCopyId from second step
- cleanup: delete the book by id from first step (hard delete)

**result:** shouldn't affect other tests


### inner-circle-books-ui

NOTE: I didn't finish writing out the tests, but the results are written

**usings**
- cypress e2e test via local-env
- prod

**prerequisite:** account with permissions
1. CanViewBooks
2. CanManageBooks
3. IsBooksHardDeleteAllowed

**1. name:** books-smoke.cy.ts

**steps:**
- authentication
- remove books where title starts with `[E2E-SMOKE]`
- check that there are no books in the list
- add book with title `[E2E-SMOKE] Новая книга`
- check that there are one book with title `[E2E-SMOKE] Новая книга`
- check its data
- remove books where title starts with `[E2E-SMOKE]`

**2. name:** take-and-return-book-flow.cy.ts

**steps:**
- authentication
- remove books where title starts with `[E2E-SMOKE]`

- remove books where title starts with `[E2E-SMOKE]`

**3. name:** book-history.cy.ts

**steps:**
- authentication
- remove books where title starts with `[E2E-SMOKE]`
- create book
- get book copy by id from get book data request
- check tracking history of this book
- return book
- remove books where title starts with `[E2E-SMOKE]`

**result:** shouldn't affect tests from other services but if we want to prallel tests inside books-ui they can be affected. To avoid this, we need to make the titles start differently in each test.

### inner-circle-documents-api

**usings**
- karate e2e test via local-env
- karate e2e test via docker-compose

**name:** documents-get-employees.feature

**prerequisite:** account with CanManageDocuments permission

**steps:**
- authentication
- get employees and macth response

**result:** shouldn't affect other tests


### inner-circle-layout-ui

**usings**
- cypress e2e test via local-env

**name:** redirect-to-employee-page.cy.ts

**prerequisite:** active account with at least ViewContacts permission for example

**steps:**
- authentication
- try to visit homepage -> redirect to employees page

**result:** shouldn't affect other tests


### inner-circle-local-env

run tests from other services in its pipelines
- inner-circle-compensations-ui
- inner-circle-books-ui
- accounts-api
- inner-circle-books-api
