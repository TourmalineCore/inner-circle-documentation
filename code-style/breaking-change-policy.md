# Breaking Changes

A breaking change is any modification that requires other services to update their code or processes to ensure continued functionality.
To keep things stable and make releases easier, we need to agree on what counts as a breaking change. We look at the whole contract, not just the API itself (endpoints), but at the whole flow, how the system is supposed to be used.

## What is a breaking change

1. Removal or modification of the contract. 
If something is removed from the public API (deleted endpoint, field in a response, required request parameter) or its type is changed.

2. Changes to mandatory scenarios. 
If you introduce a new requirement that must now be performed in the middle of an existing flow for the system to function correctly.

## What is NOT a breaking change 

1. Adding new contracts. If you add a new endpoint or a new optional field to a response/request. Old clients will continue to function using the existing logic without issues.

2. Cosmetic and internal changes. Fixing typos in documentation, internal refactoring, performance optimizations (if they don't change public behavior), updating dependencies (if they don't transitively break the public API).