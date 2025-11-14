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

- Employees Manager Role
- Books Manager Role
- Items Manager Role
- Compensations Manager Role

Do we create a "CEO" or "Manager" role with all editing permissions or do we combine several specific Manager roles in an account?

#### Destructive Roles for E2E Tests Cleanup (ALL PERMISSIONS OF SERVICE)

- AUTO_TESTS_ONLY Employees Role
- AUTO_TESTS_ONLY Books Role
- AUTO_TESTS_ONLY Items Role
- AUTO_TESTS_ONLY Compensations Role

Will these AUTO_TESTS_ONLY Roles contain all permissions of a specific service?


ToDo
How to call First and Second Tenant/Account to remember them?
Table for Compensations and Items with new Role names and which permissions they may have.
