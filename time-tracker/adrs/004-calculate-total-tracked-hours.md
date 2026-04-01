# 004: Calculate total tracked hours

## Status
Accepted (2026-04-01)

## Context
We need to make it possible for time-api to provide data about total tracked hours by project for each employee to other API services.

## Decision
Implementing a calculator in the time-api service because time-api has all the data regarding tracked hours per employee across all projects.

## Consequences
We will be able to easily calculate the total hours tracked as tasks or other entry types, and use [internal requests](../../adrs/002-internal-requests.md) to provide the necessary data to other API services. This logic will also be placed in a single location and can be scaled to meet the needs of other services.

## Alternatives

### Retrieve all entries data and calculate total in the consuming service
This is less efficient compared to the chosen solution, as it would require transferring a larger volume of data. It also makes it more difficult to reuse this functionality across other services.
