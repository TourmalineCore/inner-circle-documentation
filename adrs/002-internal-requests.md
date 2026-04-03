# 002: Internal requests

## Status
Accepted (2025-04-01)

## Context
We need to make it possible for API services to provide data from their database to other API services.

## Decision
Create internal requests that return all necessary information. These requests can be accessed by other services, such as invoices-api. Internal requests must also be secured with authorization and authentication.

## Consequences
This decision will allow internal requests to be called across other API services, reducing coupling and enabling access to required data. It also adheres to the principles of encapsulation and single responsibility, as each API should only interact with its own database, thereby preventing confusion in database operations.

## Alternatives
### Access another service's database directly
This approach could lead to security risks and tight coupling, as well as a lot of confusion when managing multiple services.