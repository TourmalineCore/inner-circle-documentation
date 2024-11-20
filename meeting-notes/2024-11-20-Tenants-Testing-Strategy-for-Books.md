# 2024-11-20 Tenants & Testing Strategy for Books

## Questions

- What is Tenant?
- How are we going to implement Tenants?
- How are we going to test feature?
- Are we going to try Event Sourcing in this service? If so then how?

## Tenants

Tenant - Organization Boundary

JWT contains TenantId => To switch a User to another Tenant we need to re-login to re-generate access token.

For instance, you login to TenantId=1 but you also are assigned to be able to access TenantId=2. This means that to switch we need to re-login and re-load the page.

Currently: Single DB for all Tenants
What we can also do:
- Every tenant has its own DB instance
pros:
a) different connection strings => isolated access per Tenant
b) each db can have different capacity/RAM/CPU
cons:
a) needs more money for each db (questionable because we might have a single DB server instance but several DB instances in the same server)
b) adding a new Tenant is a tricky process which is time consuming (at the beginning it is a manual operation)
c) migration of the DB schema is something non trivial

- Single DB instance but separate schemas per Tenant
pros:
a) different connection strings => isolated access per Tenant (per schema)
cons:
a) single generic capacity DB
b) adding a new Tenant is a tricky process which is time consuming (at the beginning it is a manual operation)
c) migration of the DB schema is something non trivial

- Every table has TenantId column
cons: we cannot split access by Tenant. If you have access to the DB then you can access everything.

Looks like multi-db and multi-schema are sort of the same from the API point of view.

Decision: use TenantId column approach for books to solve testing strategy issues first and practice it. Later for e.g. finance/salary/personal data features we try to isolated schemas per Tenant.

Inner Circle will have just a few tenants. We rarely create new Tenants.

## Tenants Testing Strategy

TDD is must have, default day-to-day working strategy.

Developers and QAs have no access to prod (k8s cluster, db).

We promote and strive to work with Local and Prod only envs.

### What and How We Test

We have User Stories. We can write E2E for each of them for all flows.
However, we need E2E tests to pass for less than 3 mins. We write E2E for only core business critical flow (no edge cases). E2E flows must be parallelized even inside a single feature.

#### E2E Core flow for books:
- Add a new book
- Take the newly created book (using single book endpoint)
- Return the book

#### Karate API Tests Flow

Let Karate tests to cover every endpoint at least once. 

- Add a new book
- Check that book is free (using books list endpoint)
- Take the newly created book
- Check that book is taken by me (using books list endpoint)
- Add a second book
- Check that the first is taken and the second is free (using books list endpoint)
- Return the first book
- Check that both books are free

### Tech Note

APIs currently run migration at startup.
- allow EF Core to work with any schema
- always upgrade db schema in a two phase update way

Need to solve this DB migration differently.

## Actions Points

- Create ADR to explain why we stick to JWT for Auth, its pros, cons, and possible alternatives for future investigation and PoCs.
- How and when do we upgrade DB schema? We want to be able to support canary releases.
- Mob session to develop API according to TDD.
- Seeding of Test data, from scratch, accumulating, etc.
