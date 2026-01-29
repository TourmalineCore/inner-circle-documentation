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

| Account Name  | Tenant                    | Role                                   | Email                     | Password   | Password Description                                                         |
| :------------ | :-----------------------: | :------------------------------------: | :-----------------------: | :--------: | :--------------------------------------------------------------------------: |
| Harry Potter  | Gryffindor                | CEO (temporary)                        | potter@tourmalinecore.com | Patronum1! | the spell to conjure a Patronus, symbolizing Harry |
| Draco Malfoy  | AUTO_TESTS_ONLY Slytherin | CEO (temporary)                        | malfoy@tourmalinecore.com | Serpens1!  | the snake-conjuring spell, symbolizing Slytherin and Draco |
| Severus Snape | AUTO_TESTS_ONLY Slytherin | CEO (temporary)                        | snape@tourmalinecore.com  | Silencio1! | the Silencing Charm, reflecting Snape's preference for controlled magic |
| Cho Chang     | AUTO_TESTS_ONLY Ravenclaw | CEO (temporary)                        | chang@tourmalinecore.com  | Reparo1!   | the mending charm, representing Cho’s responsible and caring nature |
| Gregory Goyle | AUTO_TESTS_ONLY Slytherin | AUTO_TESTS_ONLY Single Permission Role | goyle@tourmalinecore.com  | Crucio1!   | one of the Unforgivable Curses, symbolizing the dark and limited nature of Goyle |
