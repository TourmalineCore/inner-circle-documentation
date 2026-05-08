# 005: Contract‑First Approach to Feature Design with Backend and Frontend Collaboration

## Status
Proposed

## Context
One of the principles we follow when we start working on a feature is that API contract should be agreed upon before implementation, to avoid rework and late discovery of mismatches. 

We realize that with the rapid development of AI‑assisted coding, the ability to design clear contracts becomes a key skill for us. We want to intentionally shift our workflow to strengthen design skills.

Moreover, we pursue a **Test‑Driven Development (TDD) approach** in the team. It happens so that classical TDD (write test → see thats it fails → implement → refactor) is often applied only at the unit / compoment level and leaves E2E testing for the time when the functianlity is ready. We want to scale TDD further: from unit and component tests to E2E happy‑path tests, writing them *before* the implementation.

## Decision
We adopt a **five‑stage workflow** for new features invloving backend and frontend collaboration:

1. Design and agree on the contract 
2. Deploy the pure contract on the backend – no business logic, only mock implementation of endpoints for requests and responses. This unblocks frontend development.  
3. Discuss and agree on the testing strategy for the new functionality.  
4. Write happy‑path E2E tests against the agreed contract.  
5. Actually implement backend logic.

<!-- Move or replicate to the file with guidelines on processes when we have one -->
### Notes on processes
>PR review is supposed to take place within a short period of time after it was requested so that no tasks are blocked waiting for the feedback.

>Flaky tests are the top priority if there aren't any urgent tasks, they should be fixed ASAP and not put away for later.

## Consequences
### Advantages
1. Unblocks frontend at early stage.  
1. Reduces rework because contract mismatches are caught before real logic is written.  
1. Strengthens design skill.  
1. Improves discipline in writing tests because E2E happy‑path tests are written before implementation.  

### Disadvantages
1. Additional up‑front overhead. The team must agree on the contract and test design before any code is written.  
1. Mock deployment infrastructure. The backend must support mock endpoints.  
1. E2E test maintenance. Happy‑path tests written against mocks may need adjustment if the contract changes later.

## Alternatives
Contract‑first approach without mock endpoints (agree on contract but implement backend before frontend starts). E2E tests are written after implementation.  

### Advantages
1. No need to maintain mock endpoints.
1. E2E tests are more realistic since they run against the final implementation.
1. Simpler and familiar to the team: design → build backend → integrate frontend → test.

### Disadvantages
1. Frontend may be blocked. Frontend developers cannot fully test integration until backend implementation is complete.
1. Contract mismatches are discovered after backend implementation is finished, often requiring rework.
1. E2E tests simply document what was developed, this undermines TDD principles.
