# 006: Away Unpaid Entry Prototype (Frontend)

## Context
Need to design tracking of Away (Unpaid) Entry feature. 

## Folder Structure


```bash
src/
└── pages/
    └── time-tracker/
        └── sections/
            └── entry-modal/
                └── sections/
                    └── AwayUnpaidEntry
                        └── state
                            └── AwayUnpaidEntryState.ts
                            └── AwayUnpaidEntryState.cy.ts
                            └── AwayUnpaidEntryContext.ts
                        └── strategy.tsx
                        └── AwayUnpaidEntryContent.tsx
                        └── AwayUnpaidEntry.cy.tsx
                        └── AwayUnpaidEntry.scss
```

## Testing strategy

### E2E tests
Happy path steps:
- Visit time-tracker page
- Click on a free slot
- Choose Away (unpaid) type in entry modal
- Fill details (2 hours)
- Add first make-up time (1.5 hour) and second make-up time (0.5 hour)
- Save entry
- Find the created Away card and click on it
- Update Away card - change description
- Verify that both make-up time cards appeared in time-tracker

### Container tests (Entry modal)
Separate describe which will be expanded as new types will be added later (Late, Day-off (unpaid)).
Verify that updating and saving make-up time card triggers the POST method of the correct parent's endpoint.

Steps:
- Click on Make-up time entry
- Update Make-up time entry
- Verify that the related parent's endpoint is triggers

### Content tests
1. Validation test: time doesn't match -> show error message -> update -> hide error message
2. Test on disable

Steps:
- Click on Make-up time entry
- Verify that the related parent entry card is opened
- Verify that type selection, description, and start/end time of Away entry are disabled

### Tests of state
1. Init values
2. Validation test: time doesn't match -> update -> time matches
