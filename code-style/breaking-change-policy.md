# Breaking Changes

A breaking change is any modification that requires other services to update their code or processes to ensure continued functionality.
To keep things stable we need to agree on what counts as a breaking change. We look at the whole contract, not just the API itself (endpoints), but at the whole flow, how the system is supposed to be used.

## What is a breaking change

1. **Removal or modification of the contract.**
If something is removed from the public API (deleted endpoint, field in a response, required request parameter) or the type of request method is changed (e.g. PUT -> POST). 

2. **Changes to mandatory scenarios.** 
If you introduce a new requirement that must now be performed in the middle of an existing flow for the system to function correctly.

## What is NOT a breaking change 

1. **Adding new contracts.** If you add a new endpoint or a new optional field to a response/request. Old clients will continue to function using the existing logic without issues.

2. **Cosmetic and internal changes.** Fixing typos in documentation, internal refactoring, performance optimizations (if they don't change public behavior), updating dependencies (if they don't transitively break the public API).

## Changes Breaking TypeScript Contract Detected by Lint
We have a workflow that lints UI when a new js-client is published in PR so that we know that the new api changes are non-breaking from TypeScript contract point of view

1. Removing a property from a response.
Example: Removing name from ProjectDto causes TypeScript errors wherever UI code accesses this property.

```text
Error: src/pages/time-tracker/sections/entry-modal/sections/TaskEntry/TaskEntryContent.tsx(65,13): error TS2339: Property 'name' does not exist on type 'ProjectDto'.
Error: src/pages/time-tracker/sections/time-tracker-table/components/EntryContent/EntryContent.tsx(15,28): error TS2339: Property 'name' does not exist on type 'ProjectDto'.
Error: src/pages/time-tracker/sections/time-tracker-table/state/TimeTrackerTableState.cy.ts(19,9): error TS2353: Object literal may only specify known properties, and 'name' does not exist in type 'ProjectDto'.
```

2. Renaming a response that is explicitly referred by the UI.
Example: Renaming CreateUnwellEntryRequest to CreateUnwellEntryTestRequest causes import failures.

```text
Error: src/pages/time-tracker/sections/entry-modal/sections/UnwellEntry/strategy.tsx(1,10): error TS2724: '"@tourmalinecore/inner-circle-time-api-js-client"' has no exported member named 'CreateUnwellEntryRequest'. Did you mean 'CreateUnwellEntryTestRequest'?
```

Important: this is not a breaking change in HTTP contract. We simply renamed an internal class but its http contract was intact. In such case we can safely bypass this failure, merge it, and then migrate the UI to the new TypeScript types schema. That won't affect the runtime contract between API and UI.

3. Renaming query parameters or removing endpoints.
Example: Renaming startDate to startDateTest in a query parameter, and removing an endpoint like entries/{entryId}/soft-delete, causes TypeScript errors where the UI calls these functions.

```text
Error: src/pages/time-tracker/sections/entry-modal/sections/DeleteModal/DeleteModalContainer.tsx(36,15): error TS2551: Property 'trackingSoftDeleteEntry' does not exist on type '{ trackingGetEntriesByPeriod: (query: { startDateTest: string; endDate: string; }, params?: RequestParams | undefined) => Promise<AxiosResponse<GetEntriesByPeriodResponse, any, {}>>; ... 5 more ...; trackingHardDeleteEntry: (entryId: number, params?: RequestParams | undefined) => Promise<...>; }'. Did you mean 'trackingHardDeleteEntry'?
Error: src/pages/time-tracker/sections/time-tracker-table/TimeTrackerTableContainer.tsx(58,9): error TS2353: Object literal may only specify known properties, and 'startDate' does not exist in type '{ startDateTest: string; endDate: string; }'.
```

## Changes NOT Detected by TypeScript Lint
Endpoint path changes.
If we change an endpoint path, it produces no errors in lint. Even though this is a breaking change in the contract, it cannot be caught with this lint, it is expected to be caught by the ui and/or api E2E tests.

## Conventional Commits Allowed Prefixes
We need to allow only certain prefixes that lead to expected version bump and its commit.

**feat** - non-breaking functional change/new feature, minor update</br>
**feat!** - breaking change/new feature (UI can no longer call the API without adjustment), major update

**fix** - non-breaking bug fix, patch update</br>
**fix!** - breaking bug fix, major update

**refactor** - non-breaking refactoring, patch update

**format** - non-breaking code re-formatting, patch update (in case there was an accidental functionality change)

**docs** - docs change, no version bump</br>
**infra** - DevExp infrastructure update that needs no deployment, no version bump

**ci** - workflows or ci folder update that needs no deployment, no version bump</br>
**cd** - deployment update, patch update

**git** - git related changes that needs no deployment, no version bump