# Time UI Architecture

## Pages
1. **TimeTrackerPage** - main time tracker page

---

## TimeTrackerPage
Page for tracking time, containing the main time tracker components.

### Page Structure
The page consists of two sections:

#### 1. **time-tracker-table**
Handles rendering the time tracker table, where users can:
- View existing entries
- Log different events:
  - Work tasks
  - Absences
  - Overtime / compensation time
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
  - Returns a list of work entries for the given period
  - Parameters: `startDate` is the week start, `endDate` is the week end
- `GET /api/time/tracking/task-entries/projects?startDate={startDate}&endDate={endDate}`
  - Returns a list of projects for the period
  - Parameters: `startDate` is the week start, `endDate` is the week end

---

#### 2. **entry-modal**
Handles the modal window for creating and editing entries:
- Entering activity details for a time period
- Editing existing entries

**Architecture Layers:**

| Layer | Purpose |
|------|------------|
| **EntryModalState** | Manages the modal window's state |
| **EntryModalContainer** | Sends requests to create/update entries and get data |
| **EntryModalContent** | Displays the modal window |

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
