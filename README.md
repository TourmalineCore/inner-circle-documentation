# inner-circle-documentation

## Links to all project repositories
- 
- 

## Containers Diagram 

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

Container(AuthAPI, "Auth API", ".net 8 Docker-container", "Check login and password, after connect to AccountManagementAPI generate token with permissions inside.")
Container(AuthUI, "Auth UI", ".net Docker-container", "Pages for registration, authentication and password reset.")
ContainerDb(AuthDB, "Auth DB", "PostgreSQL", "Stores information for authentication: login, password hash, etc.")


Container(AccountsAPI, "Accounts API", ".net 8 Docker-container", "API contains endpoint for accounts, roles, tenants.")
Container(AccountsUI, "Accounts UI", "React", "Pages for managing accounts, roles and tenants.")
ContainerDb(AccountsDB, "Accounts DB", "PostgreSQL", "Contains accounts, roles with permissions, tenants.")


Container(InnerCircleEmailSender, "Inner Circle Email Sender", ".net 8 Docker-container", "Send emails to employees corporate mail.")
Container(Mail, "Mail.ru", "External Service", "...")


Container(InnerCircleEmployeesAPI, "Inner Circle Employees API", ".net Docker-container", "API contains endpoints for get from db information about employees...")
ContainerDb(InnerCircleEmployeesDB, "Inner Circle Employees DB", "PostgreSQL", "Stores employees information: name, GitHub/GitLab accounts, corporate and personal email, etc.")

Container(InnerCircleUI, "Inner Circle UI", "React/JS Docker-container", "Pages for managing profile and employees list.")

Container(InnerCircleCompensationsAPI, "Inner Circle Compensations API", ".net 8 Docker-container", "API contains endpoint for working with compensations.")
Container(InnerCircleCompensationsUI, "Inner Circle Compensations UI", "React", "Pages for managing and request compensations.")
ContainerDb(InnerCircleCompensationsDB, "Inner Circle Compensations DB", "PostgreSQL", "Contains information about compensation: amount, dates, tenant, etc.")


Container(InnerCircleDocumentsAPI, "Inner Circle Documents API", ".net 8 Docker-container", "API validate documents.")
Container(InnerCircleDocumentsUI, "Inner Circle Documents UI", "React", "Pages for documents upload and validation.")
ContainerDb(InnerCircleDocumentsDB, "Inner Circle Documents DB", "PostgreSQL", "Contains empty database.")


Rel(AuthUI, AuthAPI, "Request for authentication.", "http")
Rel(AuthAPI, AccountsAPI, "Get permissions for user with correct creds.", "http")
Rel(AuthAPI, AuthDB, "Read and write information.")
Rel(AuthAPI, InnerCircleEmailSender, "Send letters related with password.")

Rel(AccountsUI, AccountsAPI, "Request for account, tenant, role creation, editing, etc. Request for lists with account, tenant, role, etc.", "http")
Rel(AccountsAPI, AccountsDB, "Read and write information.")
Rel(AccountsAPI, InnerCircleEmployeesAPI, "Request for creation employee.")

Rel(InnerCircleEmployeesAPI, InnerCircleEmployeesDB, "Read and write information", "REST")

Rel(InnerCircleUI, InnerCircleEmployeesAPI, "Request information about employee, all employees, edit information in profile about employee, all employees.", "REST")


Rel(InnerCircleDocumentsUI, InnerCircleDocumentsAPI, "Request employee data and send documents files after validation.")
Rel(InnerCircleDocumentsAPI, InnerCircleEmailSender, "Send files for letters related with employee documents.")
Rel(InnerCircleDocumentsAPI, InnerCircleEmployeesAPI, "Request for personal data of employee for sending documents.")


Rel(InnerCircleCompensationsUI, InnerCircleCompensationsAPI, "Request for compensation information and managing compensations.", "http")
Rel(InnerCircleCompensationsAPI, InnerCircleCompensationsDB, "Read and write information.")
Rel(InnerCircleCompensationsAPI, InnerCircleEmployeesAPI, "Get information about employees and them compensations.")

Rel(InnerCircleEmailSender, Mail, "Send letters.", "SMTP")


SHOW_LEGEND()
@enduml
```
