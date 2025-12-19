# Tenants, Roles, Accounts Automation Strategy

TBD - To Be Done

Strategic goal is to run the same E2E tests of all services against Local Env and against Prod Env.
It has to support parallel execution of all E2E tests against an environment. They shouldn't conflict. There should be no race condition.

## Local Env

### Tenants

- Developer Tenant
- AUTO_TESTS_ONLY First Tenant
- AUTO_TESTS_ONLY Second Tenant

### Accounts in Each Tenant

- Manager Account
- First Employee Account
- Second Employee Account
- No Permissions Account

### Roles

#### Viewing as Employee and Editing their own data

- Employee

#### Manager Roles Per Service (Can Edit Data)

- Employees Manager
- Books Manager
- Items Manager
- Compensations Manager

Normally each of these roles contain all permissions of their service except for `AUTO_TESTS_ONLY` ones. For instance, `AUTO_TESTS_ONLY Items` contains all permissions of Items service except for `AUTO_TESTS_ONLY_IsItemTypesHardDeleteAllowed` and `AUTO_TESTS_ONLY_IsItemsHardDeleteAllowed`. There might be also some permissions missing for a Manager Role depending on the service needs.

#### Destructive Roles for E2E Tests Cleanup (ALL PERMISSIONS OF SERVICE)

- AUTO_TESTS_ONLY Employees
- AUTO_TESTS_ONLY Books
- AUTO_TESTS_ONLY Items
- AUTO_TESTS_ONLY Compensations

Each of these roles contain all permissions of their service. For instance, `AUTO_TESTS_ONLY Items` contains all permissions of Items service.

ToDo
How to call First and Second Tenant/Account to remember them?
Table for Compensations and Items with new Role names and which permissions they may have.
