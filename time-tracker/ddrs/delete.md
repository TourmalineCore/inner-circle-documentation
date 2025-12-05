# Delete

## Status
Proposed (2025-12-04)

## Context
We need to specify a reason why an entry was deleted, in order to make it clear why the event was created and then not just edited in case of inaccuracy, but deleted. It is implied that an employee had a plan and it didn't come true, and if we don't specify the reason, it won't be clear why. In this case the manager will have trouble understanding why an employee worked when they were not supposed to, or didn't work when they were supposed to.    

![image info](./images/delete-context.png)

## Decision

### Delete cases for all events

### Task

![image info](./images/delete-task.png)

### Deleting Make-up time in the Away event, before saving

![image info](./images/delete-make-up-time-1.png)

### Deleting Make-up time in the Away event, after saving

![image info](./images/delete-make-up-time-2.png)

### Deleting Make-up time in the Make-up time event
It won't be there before saving because we don't add the Make-up time by itself, it's added from the Away event

![image info](./images/delete-make-up-time-3.png)

### Away
There's no case before saving here, since if we need to delete the Away event before saving, we can just close the event.

![image info](./images/delete-away-1.png)

![image info](./images/delete-away-2.png)

### Sick leave (the same logic applies to vacation and day-off)

![image info](./images/delete-sick-leave.png)

## Alternatives

### Button
Dots are less clear, more clicks needed to delete, but we can lay down the logic for the future when Copy functionality appears.

![image info](./images/delete-option-1.png)

### Cancel delete
After deleting a task, a confirmation bar appears to confirm the action performed with the option to Undo delete. This is good from the UX perspective to let the user undo a deletion made by mistake, but at the moment the user is well insured against accidental deletion by the confirmation box. It's also technically difficult to reconstruct the chain of deletions. We can think this through better at a later stage, we won't do it now. 

![image info](./images/delete-option-2.png)
