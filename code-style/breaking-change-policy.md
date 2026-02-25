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