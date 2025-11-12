# Local-env tenants and accounts

## Tenants list

List of tenants that are currently created

- TourmalineCore
- Test


## Accounts list

List of accounts that are currently created

| Account Name      |     Email                             |     Password       |     Tenant     | Permissions | Tests |           Comment          |
| :---------------- | :-----------------------------------: | :----------------: | :------------: | :---------: | :---: | :------------------------: |
| Trial Trial Trial | trial@tourmalinecore.com              | Trial999*PassWord3 | TourmalineCore |  CEO (all)  |   -   | admin in prod and local-env |
| Admin Admin Admin | inner-circle-admin@tourmalinecore.com | *unknown*          | TourmalineCore | Admin (all) |   -   | prod admin, it's also in local-env, but hidden (IsSpecial = true) |
| Ceo Ceo Ceo       | ceo@tourmalinecore.com                | cEoPa$$wo1d        | TourmalineCore |  CEO (all)  |   *   | local-env admin and account through which all tests are run |
<!-- | One Test1 User    | test1@tourmalinecore.com              | Test123!5673Th3    |      Test      |  CEO (all)  |   -   | account created by new job |
| Two Test2 User    | test2@tourmalinecore.com              | Test123!5673Th3    |      Test      |  CEO (all)  |   -   | account created by new job | -->

*Tests and services where its used
- accounts-api
- auth-api
- auth-ui
- inner-circle-compensations-api
- inner-circle-compensations-ui
- inner-circle-books-api
- inner-circle-books-ui
- inner-circle-documents-api
- inner-circle-layout-ui
- inner-circle-local-env
- inner-circle-ui - set in cypress config but not used
