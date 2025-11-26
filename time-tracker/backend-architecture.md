# Backend Architecture

## What is architecture for?

- A mental model that helps the developer navigate a large codebase
- Divides the application into separate parts with their own concerns
- Limits the propagation of code changes
- Allows to faster discover needed functionality
- Allows to reuse the lower-level layers code (e.g., access to the database) at higher levels

## 3-Layer Architecture

- Presentation Layer (UI)
- Business Logic Layer (BLL)
- Data Access Layer (DAL)

```
Presentation Layer
     ↓
Business Logic Layer
     ↓
Data Access Layer
```

## Presentation Layer

- Implements interaction with the consumer (HTTP, gRPC, Telegram API, Console, UI)
- Performs basic input data validation (checking email format, field completion)
- Deserializes the request and serializes the response
- Calls the Business Logic Layer

## Business Logic Layer (Application Layer)

- Implements the rules (business logic) of the system
- Performs more complex input data validation
- Coordinates calls to the Data Access Layer, aggregates data from different data sources
- Does not know about the details of UI or database implementation

**Example:** `CreateClientHandler` (`CreateClientCommand`) contains the logic for creating a client, checking for duplicates, sending notifications

## Data Access Layer (DAL, Infrastructure Layer)

- Implements data access by abstracting data sources from the rest of the system
- Works with databases, caches, file systems, and external APIs
- Data sources can include:
  - relational databases (PostgreSQL, MySQL, MS SQL)
  - non-relational databases (Redis, MongoDB, ClickHouse)
  - other services
  - audit systems

## Layers Dependencies

- Lower-level layers know nothing about the layers above them (often implemented through separation into projects and adding links between them)
- Upper-level layers can only use the layer below them (the Presentation layer should not use the DAL directly, only through the Application layer)

```
Presentation Layer
     ↓
Business Logic Layer
     ↓
Data Access Layer
```

## Layers and their Characteristics

| Layer               | Responsibilities                        | Components                                 | Knows about                  | Doesn't know about                  |
| ------------------ | -------------------------------------- | ------------------------------------------ | ------------------------ | --------------------------- |
| **Presentation**   | Request acceptance, validation, serialization | Controller, HttpContext, Request, Response | BLL Interfaces           | BLL and DAL Implementation     |
| **Business Logic** | Rules implementation, coordination         | Handler, Service, Validator                | DAL Interfaces, Entities  | Database details, HTTP            |
| **Data Access**    | CRUD operations, storage abstraction    | Repository, DbContext, Query Builder       | Entity model             | HTTP, UI, Application rules |

## Layer Terminology

| Layer               | Accepts           | Returns          | Which classes are involved                            |
| ------------------ | ----------------------- | ----------------------- | ------------------------------------------------- |
| **Presentation**   | HTTP Request, Request   | HTTP Response, Response | Controller, HttpContext, Middleware, Filter, View |
| **Business Logic** | Request, Query, Command | Response, Result, Dto   | Service, Handler, Command/Query                   |
| **Data Access**    | Entity                  | Entity                  | Repository, DbContext                             |

---

## 3-Layer Architecture vs Clean Architecture

- 3 Layer - Presentation => Application => Data Access Layer

  - Easier to understand
  - More difficult to perform unit testing because the Application Layer depends on the Data Access Layer 
    - Integration tests can be used instead
  - Makes it easier to use the capabilities of the database

- Clean Architecture - Presentation => Application <= Data Access Layer
  - Also known as Hexagonal Architecture, Ports-and-Adapters, Onion Architecture
  - Often used in conjunction with the Rich Domain Model
  - Abstracts data access through interfaces implemented in the DAL =>
    - simplifies writing unit tests
    - _data storage method can be easily changed_
  - Additional abstractions add complexity and make understanding more difficult

## 3-Layer vs Clean Architecture

### 3-Layer Architecture

- Easier to understand
- Faster to develop
- More difficult to perform unit testing, since the Application Layer depends on the Data Access Layer
  - Integration tests can be used instead of unit tests
- It is easy to violate separation of concerns
- Business logic is often mixed with data storage (which can be both a benefit and a drawback)

**Direction of dependencies:**

```
Presentation → Business Logic → Data Access
```

### Clean Architecture (Hexagonal Architecture, Ports-and-Adapters, Onion Architecture)

- Abstracts data access through interfaces implemented in the DAL
  - simplifies writing unit tests
- _data storage method can be easily changed_
- Business logic is easier to test (unit tests without a database)
- Clear boundaries between layers
- Additional abstractions add complexity and make understanding more difficult

**Direction of dependencies (inversion):**

```
Presentation → Business Logic (Application Core) ← Data Access (Infrastructure)
```

## Approaches to Organizing Business Logic (Patterns of Enterprise Application Architecture by Martin Fowler)

### 1. Transaction Script

**Description:** All transaction logic is stored in one place (method/procedure). Each script is responsible for one business operation.

Can be implemented at both the database level (stored procedures) and the application level

**Example:** `CreateClientHandler`, `UpdateOrderCommand`, `PaymentService`

**Advantages:**

- ✅ The simplest option
- ✅ Logic in one place, no need to jump between files

**Disadvantages:**

- ❌ Can lead to duplicate logic between scripts
- ❌ More difficult to test (often requires a database)

### 2. Active Record

**Description:** An entity combines data and the logic for storing it.

Not used in real development (https://www.castleproject.org/projects/activerecord)

### 3. Table Module

All the logic for working with a single table is contained in a single object.

### 4. Rich Domain Model 🎯

**Description:** An entity contains validation, business rules, and invariants. Methods ensure that the object is always in a valid state.

**Example:**

```csharp
public class Client
{
    private ClientStatus _status;

    public void SetStatus(ClientStatus newStatus)
    {
        // Business rule: you can only downgrade your status by 1 level
        if (newStatus < _status - 1)
            throw new InvalidStatusTransitionException();

        _status = newStatus;
    }

    public Client(string name, string email)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("Name cannot be empty");
        if (!email.Contains("@"))
            throw new ArgumentException("Invalid email");

        Name = name;
        Email = email;
    }
}
```

**Advantages:**

- Pure business logic separated from infrastructure
- Easy to write unit test (no database, no mocks)

**Disadvantages:**

- Requires more design time
- May be an overkill for simple CRUD applications
- Requires a good understanding of the domain
- Low performance - some checks are more efficient and easier to implement at the database level

**Remember:** The best architecture is the one you understand and can explain to a colleague.

## Useful Links

- https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures
- https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles
- https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-10.0&pivots=xunit
- https://learn.microsoft.com/en-us/azure/architecture/
