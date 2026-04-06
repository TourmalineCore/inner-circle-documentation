# 001: Null object for current entry

## Status
Proposed

## Context
In `EntryModalState`, there is a _currentEntry field, which by default may be empty (a new entry) and may also contain an existing entry with filled fields. Currently, an empty record is designated `null`, which entails additional checks in the code for null or the use of `!`and `?` when accessing the _currentEntry fields.

```typescript
type TrackedEntry = {
  id?: number,
  title?: string,
  taskId?: string,
  project?: ProjectDto,
  description?: string,
  type?: EntryType,
  date: Date,
  start: Date,
  end: Date,
}

private _currentEntry: TrackedEntry | null = null
```

## Decision
Use null object pattern and create an object with default values for _currentEntry and slightly change the type of `TrackedEntry`:

```typescript
type TrackedEntry = {
  id?: number,
  title?: string,
  taskId?: string,
  project?: ProjectDto,
  description?: string,
  type?: EntryType,
  // Adding the null type
  date: Date | null,
  start: Date | null,
  end: Date | null,
}

const EMPTY_ENTRY: TrackedEntry = {
  title: ``,
  taskId: ``,
  description: ``,
  type: EntryType.TASK,
  date: null,
  start: null,
  end: null,
}

private _currentEntry: TrackedEntry = EMPTY_ENTRY
```

## Consequences
Advantages:
1. Allows you to get rid of null checks 
2. Fewer uses of the signs `!`and `?` when accessing fields in _currentEntry
3. Allows you to minimize the occurrence of the error `Cannot read property of null`
4. The apparent absence of a date

Disadvantages:
1. `TrackedEntry` type pollution

## Alternatives
### Keep null as the default value
Advantages:
1. The apparent absence of _currentEntry
2. Clean `TrackedEntry` type

Disadvantages:
1. Null checks are required.
2. Risks of the error `Cannot read property of null`
3. A large number of characters `!`and `?` when accessing fields in _currentEntry

### Do not add null type for date, start and end
```typescript
type TrackedEntry = {
  id?: number,
  title?: string,
  taskId?: string,
  project?: ProjectDto,
  description?: string,
  type?: EntryType,
  date: Date ,
  start: Date,
  end: Date,
}

const EMPTY_ENTRY: TrackedEntry = {
  title: ``,
  taskId: ``,
  description: ``,
  type: EntryType.TASK,
  // Using a fake date
  date: new Date(0),
  start: new Date(0),
  end: new Date(0),
}

private _currentEntry: TrackedEntry = EMPTY_ENTRY
```

Advantages:
1. Allows you to get rid of null checks 
2. Fewer uses of the signs `!`and `?` when accessing fields in _currentEntry
3. Minimizes the occurrence of the error `Cannot read property of null`
4. Clean `TrackedEntry` type

Disadvantages:
1. The implicit absence of a date
2. The risk of using an invalid date (1970)

### Make date, start, and end optional in the type
```typescript
type TrackedEntry = {
  id?: number,
  title?: string,
  taskId?: string,
  project?: ProjectDto,
  description?: string,
  type?: EntryType,
  date?: Date ,
  start?: Date,
  end?: Date,
}

const EMPTY_ENTRY: TrackedEntry = {
  title: ``,
  taskId: ``,
  description: ``,
  type: EntryType.TASK,
}

private _currentEntry: TrackedEntry = EMPTY_ENTRY
```

Advantages:
1. Allows you to get rid of null checks 
2. Fewer uses of the signs `!`and `?` when accessing fields in _currentEntry
3. Minimizes the occurrence of the error `Cannot read property of null`
4. The apparent absence of a date

Disadvantages:
1. Forced optional types in `TrackedEntry`