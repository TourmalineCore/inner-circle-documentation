# DDR: Creation of Test Tenants, Roles, and Accounts

## Context
We need to create test data:
- **Tenants** based on Harry Potter houses
- **Roles** with different permission sets
- **Accounts** Harry Potter characters with passwords linked to spells

## Decisions

### 1. Tenants

| Tenant Name               | Description                |
| :------------------------ | :------------------------: |
| Gryffindor                | primary tenant for develop |
| AUTO_TESTS_ONLY Slytherin | first tenant for testing   |
| AUTO_TESTS_ONLY Ravenclaw | second tenant for testing  |


### 2. Roles

| Role Name                              | Description                                                                                               |
| :------------------------------------- | :-------------------------------------------------------------------------------------------------------: |
| AUTO_TESTS_ONLY Single Permission Role | temporary role with one permission, in the future its planned to be a role without any permissions at all |


### 3. Accounts

| Account Name  | Tenant                    | Role                                   | Password | Password Description                                                         |
| :------------ | :-----------------------: | :------------------------------------: | :------: | :--------------------------------------------------------------------------: |
| Harry Potter  | Gryffindor                | CEO (temporary)                        | lumos    | the light spell, symbolizing Harry                                           |
| Draco Malfoy  | AUTO_TESTS_ONLY Slytherin | CEO (temporary)                        | serpens  | the snake-conjuring spell, symbolizing Slytherin and Draco Malfoy            |
| Cho Chang     | AUTO_TESTS_ONLY Ravenclaw | CEO (temporary)                        | avis     | the bird-conjuring spell, symbolizing lightness and freedom                  |
| Gregory Goyle | AUTO_TESTS_ONLY Slytherin | AUTO_TESTS_ONLY Single Permission Role | nox      | the spell to extinguish light, opposite of lumos, symbolizing no permissions |
