# Time UI Architecture

## Pages
1. **TimeTrackerPage** - main time tracker page

---

## TimeTrackerPage
Page for tracking time, containing the main time tracker components.

### Page Structure
The page consists of two sections:

#### 1. **time-tracker-table**
This layer pass data on current entry that is being created to EntryModal section.<br/>
It handles rendering of the time tracker table with various entry types:
  - Tasks
  - Absences
  - Overtime 
  - Make-up time
  - Vacation
  - Sick leave, etc.

**Architecture Layers:**

| Layer | Purpose |
|------|------------|
| **TimeTrackerTableState** | Manages the section's state |
| **TimeTrackerTableContainer** | Sends requests to get data |
| **TimeTrackerTableContent** | Displays the table |

**API Endpoints (TimeTrackerTableContainer):**
- `GET /api/time/tracking/entries?startDate={startDate}&endDate={endDate}`
  - Returns a list of entries for the given period
  - Parameters: `startDate` is the week start, `endDate` is the week end
- `GET /api/time/tracking/task-entries/projects?startDate={startDate}&endDate={endDate}`
  - Returns a list of projects for the period
  - Parameters: `startDate` is the week start, `endDate` is the week end

---

#### 2. **entry-modal**
Handles the modal window for creating and editing entries:
- Entering activity details for a time period
- Updating existing entries
- Copying existing entries
- Deleting existing entries

**Architecture Layers:**

| Layer | Purpose |
|------|------------|
| **EntryModalState** | Manages the modal window's state |
| **EntryModalContainer** | Sends requests to create/update entries and get data |
| **EntryModalContent** | Displays the modal window |

There are various entry types (e.g., task entry, unwell entry), each with its own state and content.

**API Endpoints:**

- `GET /api/time/tracking/entries/startDate={startDate}&endDate={endDate}`
  - Get a list of all entries for a period starting with startDate and ending on endDate
- `GET /api/time/tracking/task-entries/projects?startDate={startDate}&endDate={endDate}`
  - Get a list of projects for a single day
  - Parameters: `startDate` and `endDate` take the same date, because we only need projects for one day
- `POST /api/time/tracking/task-entries`
  - Create a new task entry
- `POST /api/time/tracking/task-entries/{taskEntryId}`
  - Update an existing work entry
- `POST /api/time/tracking/unwell-entries`
  - Create a new unwell entry
- `POST /api/time/tracking/unwell-entries/{unwellEntryId}`
  - Update an existing unwell entry
- `DELETE /api/time/tracking/entries/{entryId}/soft-delete`
  - Delete an existing entry

## Strategy Pattern
We have a number of entry types which have alike fields and methods. To avoid a lot of duplication, we use the strategy pattern which allows to define repeated parts, encapsulate each one in a separate object, and make them interchangeable.

This is the **type** we have for **EntryStrategy**:

```typescript
export type EntryStrategy = { 
  EntryContent: ReactNode,
  validateOnClient: ({
    entryState,
  }: {
    entryState: any,
  }) => boolean,
  createEntryAsync: ({
    requestData,
  }: {
    requestData: any,
  }) => Promise<unknown>,
  updateEntryAsync: ({
    id,
    requestData,
  }: {
    id: number,
    requestData: any,
  }) => Promise<unknown>,
}
```
This the implementation of the strategy for the **task** entry type:
```typescript
export const TASK_ENTRY_STRATEGY: EntryStrategy = {
  EntryContent: <TaskEntryContent />,
  validateOnClient: ({
    entryState,
  }: {
    entryState: TaskEntryState,
  }) => validateTaskEntry({
    entryState, 
  }),
  createEntryAsync: ({
    requestData,
  }: {
    requestData: CreateTaskEntryRequest,
  }) => api.trackingCreateTaskEntry(requestData),
  updateEntryAsync: ({
    id,
    requestData,
  }: {
    id: number,
    requestData: UpdateTaskEntryRequest,
  }) => api.trackingUpdateTaskEntry(id, requestData),
}
```
This the implementation of the strategy for the **unwell** entry type:

```typescript
export const UNWELL_ENTRY_STRATEGY: EntryStrategy = {
  EntryContent: <UnwellEntryContent />,
  validateOnClient: () => true,
  createEntryAsync: ({
    requestData,
  }: {
    requestData: CreateUnwellEntryRequest,
  }) => api.trackingCreateUnwellEntry(requestData),
  updateEntryAsync: ({
    id,
    requestData,
  }: {
    id: number,
    requestData: UpdateUnwellEntryRequest,
  }) => api.trackingUpdateUnwellEntry(id, requestData),
}
```

This is how all strategies are united in one object:

```typescript
export const ENTRY_TYPES_STRATEGY: Record<EntryType, EntryStrategy> = {
  [EntryType.TASK]: TASK_ENTRY_STRATEGY,
  [EntryType.UNWELL]: UNWELL_ENTRY_STRATEGY,
}
```

This is how the strategy is used in the component in the submit method:
```typescript
  async function onSubmitEntry() {
    const entryStrategy = ENTRY_TYPES_STRATEGY[type]

    const isValid = entryStrategy.validateOnClient({
      entryState,
    })

    if (!isValid) {
      return
    }
    
    try {
      const requestData = ...,

      if (id) {
        await entryStrategy.updateEntryAsync({
          id,
          requestData,
        })
      }
      else {
        await entryStrategy.createEntryAsync({
          requestData,
        })
      }
    }
    catch () {}
  }
```

## Event-bus Pattern
The time-tracker UI uses event-driven architecture for managing cross-component communication. This approach was chosen to avoid props drilling, leave the sections loosely coupled and coordinate opening modals, refreshing data, etc.

All events are listed in **EventBusMap type**, and these predefined events can be triggered or subscribed to:

```typescript
type EventBusMap = {
  'ENTRY_MODAL:OPEN': unknown,
  'ENTRY_MODAL:CLOSE': unknown,
  ...
}
```

The **EventBus class** manages event subscriptions and triggers using this Map of event names.

For component-specific state or parent-child communication, we still use React's local state and props. The event bus complements React's component model, but doesn't replace it.

### Usage Examples
#### Triggering Events
Components can trigger events without knowing which components are listening:
```typescript
  const openEntryModalEvent = () => eventBus.trigger('ENTRY_MODAL:OPEN') 

  <button onClick={openEntryModalEvent}>
```

#### Subscribing to Events
Components can subscribe to events and clean up when unmounting and resubsribe when isOpenModal state changes:
```typescript
  const [
    isOpenModal,
    setIsOpenModal,
  ] = useState(false) 

  useEffect(() => {
    const unsubscribeEntryModalOpen = eventBus.subscribe(`ENTRY_MODAL:OPEN`, () => {
      setIsOpenModal(true)
    })
     
    return () => {
      unsubscribeEntryModalOpen()
    }
  }, [
    isOpenModal,
  ])
```

  