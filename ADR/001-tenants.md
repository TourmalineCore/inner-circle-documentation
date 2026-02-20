# 001:Tenants ADR  
## What is Tenant?
Organization/Client to split data between them.

## Why?
- One service can be used by different organizations.
- Organizations cannot access and affect other orgs data.

## How?

### Single DB store all tenants data devided by TenantId column

#### Pros:
- It is easy to access data;
- Don't need to create db each time or a connection string.

#### Cons:
- Single db is as big as one tenant multiplied by its size. Relatively too much data;
- Easy to forget to filter by TenantId;
- You can accidentally drop other's data. For instance, cascade delete;
- Single point of failure. If DB isn't accessible all clients can't work;
- Harder to grant access per tenant. 
#### Questions:
- Performance to filter by tenantId all the time in all tables? 
- Need to have at least indexes?
- Many some views by tenantId and tenants' accounts which can access only views of their tenant?

### Each tenant has its own DB instance
#### Pros:
- Harder to make a mistake at accidental removing data or giving access to another tenant's data;
- Natively manage acceess granually. For instance, devs cannot access real clients' tenants;
- Potentially outage of one client doesn't mean denial of service for others.

#### Cons:
- When creating a new tenant we need to "manually" create a new db instance;
- So many DBs.

#### Questions:
- How do we switch connection strings at runtime? 
- Who keeps this knowledge and provide it? 
- Making creds to be the same?  
- How to make connection strings different?

## Why do we want to make our own single Client Inner Circle to be a Multi-Tenant app?
- Split Access. Isolated access to data. For instance, developers cannot access real production data;
- Testability. Separate Tenants for tests execution and users (at least);
- Prod Only Vision. Instead of Dev and Prod envs where we don't need Multi-Tenancy;
- We have an open-source showcase and example => gain practical experience.

## What do we have now about Multi-Tenancy?
- Can manage tenants;
- Testing tenant where against which we execute tests.

## Which tenants we need?
- Tourmaline Core - main, with real Employees;
- TESTE2E Pipelines - tenant which has Accounts for E2E tests execution against Prod.

## Naming Conventions
- All tests related Tenants starts with TEST;
- All tests related only Roles/Permissions startes with TEST_.