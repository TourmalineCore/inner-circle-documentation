# 005: Backlog Task State Tracking

## Status
Accepted (2026-07-03)

## Context
We need a clear and consistent way to track the state of backlog items (tasks, user stories, bugs) during the refinement and estimation process. 

## Decision

We have identified 7 states that a backlog item may pass through:

1. **_needs description_**: item has been created but not described
2. **_needs review_**: item has been described but not reviewed with the team 
3. **_needs refinement_**: item has been reviewed but sent back for refinement 
4. **_needs clarification_**: item has been reviewed but requires clarification with the product owner / management 
5. **_needs estimation_**: item has been agreed with the team and is ready for estimation
6. **_sprint-ready_**: item has been approved with the team and is ready for the sprint
7. **_blocked_**: item has been reviewed and is blocked

Each state is represented as a **label** on the GitHub board. Labels are applied to the task directly.
It is possible that an item has more than one label (e.g., needs estimation & blocked, etc.)

### Advantage
Labels are filterable, searchable and visible in list views.

### Disadvantage
Requires discipline to maintain relevant labels. Labels may be accidentally added or removed.

## Alternative
Custom GitHub board statuses. 

### Advantage
Make it possible to setup certain workflow to track transitions automatically.

### Disadvantage
Status names are not visible in the task summary view without opening the task. 
