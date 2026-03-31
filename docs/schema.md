# Task Manager — Data Schema & File Structure

## Task Data Schema

### Task Object Shape

```js
{
  id:          number,   // Positive integer, auto-incremented from 1
  title:       string,   // Required, 1–200 characters, trimmed
  description: string,   // Optional, 0–1000 characters, trimmed (default: '')
  status:      string,   // Enum — see Status Values below (default: 'todo')
  priority:    string,   // Enum — see Priority Values below (default: 'medium')
  createdAt:   string,   // ISO 8601 timestamp, set on creation, never mutated
  updatedAt:   string,   // ISO 8601 timestamp, set on creation, updated on every write
}
```

### Field Reference

| Field         | Type   | Required | Default    | Constraints                         |
|---------------|--------|----------|------------|-------------------------------------|
| `id`          | number | auto     | —          | Positive integer, unique, read-only |
| `title`       | string | yes      | —          | 1–200 chars, trimmed                |
| `description` | string | no       | `''`       | 0–1000 chars, trimmed               |
| `status`      | string | no       | `'todo'`   | One of the status enum values       |
| `priority`    | string | no       | `'medium'` | One of the priority enum values     |
| `createdAt`   | string | auto     | —          | ISO 8601, immutable after creation  |
| `updatedAt`   | string | auto     | —          | ISO 8601, updated on every mutation |

### Validation Rules

Validation is performed in `taskManager.js` before any storage operation. `storage.js` trusts its callers and performs no validation of its own.

#### `id`

| Rule | Detail |
|------|--------|
| Type | Must be a positive integer (`Number.isInteger(id) && id > 0`) |
| Presence | Auto-assigned by `storage.js`; never accepted as user input on create |
| On read/update/delete | Must refer to an existing task; throws if not found |
| Error message | `'Task with id <id> not found'` |

#### `title`

| Rule | Detail |
|------|--------|
| Required | Must be present and non-empty on create |
| Type | Must be a non-empty string after trimming whitespace |
| Min length | 1 character (after trim) |
| Max length | 200 characters (after trim) |
| Stored value | Trimmed string |
| Error — missing | `'title is required'` |
| Error — too long | `'title must be 200 characters or fewer'` |

#### `description`

| Rule | Detail |
|------|--------|
| Required | No — defaults to `''` if omitted |
| Type | String (if provided) |
| Max length | 1000 characters (after trim) |
| Stored value | Trimmed string, or `''` if not provided |
| Error — wrong type | `'description must be a string'` |
| Error — too long | `'description must be 1000 characters or fewer'` |

#### `status`

| Rule | Detail |
|------|--------|
| Required | No — defaults to `'todo'` on create |
| Allowed values | `'todo'` \| `'in-progress'` \| `'done'` |
| Case sensitive | Yes — lowercase only |
| Error — invalid | `'status must be one of: todo, in-progress, done'` |

#### `priority`

| Rule | Detail |
|------|--------|
| Required | No — defaults to `'medium'` on create |
| Allowed values | `'low'` \| `'medium'` \| `'high'` |
| Case sensitive | Yes — lowercase only |
| Error — invalid | `'priority must be one of: low, medium, high'` |

#### `createdAt` / `updatedAt`

| Rule | Detail |
|------|--------|
| Set by | `taskManager.js` using `new Date().toISOString()` |
| User input | Never accepted from the caller — always system-generated |
| `createdAt` | Set once on creation; never mutated afterwards |
| `updatedAt` | Set on creation; overwritten on every `updateTask` call |

#### Update Operations

Only the following fields are accepted in an `updateTask` call: `title`, `description`, `status`, `priority`. Any unrecognised field is silently ignored. Each provided field is validated against the rules above before the update is applied. `id`, `createdAt`, and `updatedAt` cannot be changed by the caller.

#### General Rules

- Throw an `Error` object (never a plain string) for every violation.
- Validate all fields before touching storage — no partial writes.
- Strip leading/trailing whitespace from string inputs before length checks and storage.

---

### Enum Values

#### Status

```js
['todo', 'in-progress', 'done']
```

| Value         | Meaning                          |
|---------------|----------------------------------|
| `'todo'`      | Task has not been started        |
| `'in-progress'` | Task is actively being worked on |
| `'done'`      | Task has been completed          |

#### Priority

```js
['low', 'medium', 'high']
```

| Value      | Meaning                                      |
|------------|----------------------------------------------|
| `'low'`    | Nice to have; can be deferred                |
| `'medium'` | Standard priority (default)                  |
| `'high'`   | Urgent or blocking; should be resolved first |

### Priority Sort Order

When sorting by priority, use this ordinal mapping (highest first):

```js
{ high: 3, medium: 2, low: 1 }
```

### Example Task Object

```js
{
  id: 1,
  title: 'Write unit tests',
  description: 'Cover taskManager.js with edge cases',
  status: 'in-progress',
  priority: 'high',
  createdAt: '2026-03-31T10:00:00.000Z',
  updatedAt: '2026-03-31T11:30:00.000Z',
}
```

---

## In-Memory Data Structure

Tasks are stored in a plain JavaScript `Map` keyed by task id, plus a counter for id generation.

```js
// Internal state in storage.js
const tasks = new Map();   // Map<number, Task>
let nextId = 1;            // Auto-increment counter
```

### Rationale for `Map` over `Array`

| Concern        | `Map`                             | `Array`                         |
|----------------|------------------------------------|---------------------------------|
| Lookup by id   | O(1)                               | O(n) scan                       |
| Delete by id   | O(1)                               | O(n) splice                     |
| Ordered list   | Iterate insertion order            | Native index order              |
| Memory         | Comparable                         | Comparable                      |

Listing and filtering operations convert to an array via `[...tasks.values()]`.

---

## File Structure

```
task-manager/
├── bin/
│   └── task-manager.js      # Executable entry point (sets up CLI, invokes cli.js)
├── src/
│   ├── cli.js               # Argument parsing and command routing
│   ├── taskManager.js       # Business logic — validation, filtering, sorting
│   ├── storage.js           # In-memory data store — raw CRUD on the Map
│   ├── constants.js         # Enums and validation limits
│   └── utils.js             # Formatting and display helpers
├── docs/
│   ├── project-plan.md      # Project plan
│   └── schema.md            # This file
├── package.json             # Project metadata and bin entry
└── README.md                # User-facing usage documentation
```

---

## Module Responsibilities

### `bin/task-manager.js`

- Marks the file executable (`#!/usr/bin/env node`)
- Reads `process.argv`
- Delegates to `cli.js`
- Catches unhandled errors and exits with code 4

### `src/cli.js`

- Parses raw `process.argv` arguments using Node.js built-ins
- Routes to the correct command handler (add, list, update, delete)
- Prints user-facing output and error messages
- Sets `process.exitCode` on failure
- Imports from: `taskManager.js`, `utils.js`

### `src/taskManager.js`

- Validates all inputs before any storage operation
- Constructs new task objects (assigns id, sets timestamps)
- Provides: `createTask`, `getTask`, `listTasks`, `updateTask`, `deleteTask`
- Provides: `filterTasks`, `sortTasks`
- Throws `Error` with descriptive messages on invalid input
- Imports from: `storage.js`, `constants.js`

### `src/storage.js`

- Owns the `Map` and `nextId` counter
- Provides raw CRUD: `insert`, `findById`, `findAll`, `update`, `remove`
- No validation — trusts callers (taskManager.js) to validate first
- Exports nothing that leaks internal state directly (returns copies)

### `src/constants.js`

- Exports `STATUSES`, `PRIORITIES`, and validation limits

```js
export const STATUSES   = ['todo', 'in-progress', 'done'];
export const PRIORITIES = ['low', 'medium', 'high'];

export const PRIORITY_ORDER = { high: 3, medium: 2, low: 1 };

export const DEFAULTS = {
  status:      'todo',
  priority:    'medium',
  description: '',
};

export const LIMITS = {
  titleMin:       1,
  titleMax:       200,
  descriptionMax: 1000,
};
```

### `src/utils.js`

- Formats task objects for human-readable terminal output
- Provides table or aligned-text display helpers
- No business logic; pure display functions
- Imports from: `constants.js`

---

## Module Dependency Graph

```
bin/task-manager.js
    └── src/cli.js
            ├── src/taskManager.js
            │       ├── src/storage.js
            │       └── src/constants.js
            └── src/utils.js
                    └── src/constants.js
```

No circular dependencies. `storage.js` and `constants.js` have no internal imports.

---

## Public API Summary

### `src/taskManager.js` Exports

```js
createTask({ title, description?, priority? })         → Task
getTask(id)                                            → Task
listTasks()                                            → Task[]
updateTask(id, { title?, description?, status?, priority? }) → Task
deleteTask(id)                                         → void
filterTasks({ status?, priority? })                    → Task[]
sortTasks(tasks, { by: 'priority'|'createdAt', order?: 'asc'|'desc' }) → Task[]
```

### `src/storage.js` Exports

```js
insert(task)        → Task        // Assigns id, stores, returns stored task
findById(id)        → Task|null
findAll()           → Task[]      // Returns shallow copy of all tasks
update(id, fields)  → Task        // Merges fields, returns updated task
remove(id)          → boolean     // Returns true if deleted, false if not found
clear()             → void        // Resets store (useful for tests)
```

---

## Error Handling Strategy

### Principles

1. **Throw `Error` objects** — never throw plain strings. Every `throw` must use `new Error('...')`.
2. **Validate before writing** — all inputs are validated in `taskManager.js` before any storage operation. No partial writes.
3. **Fail fast** — invalid inputs throw immediately; callers should not need to inspect return values to detect failure.
4. **User-facing messages** — `cli.js` catches errors and formats them for the terminal. Raw stack traces are never shown to users in normal operation.
5. **Log with `console.error`** — errors are written to stderr, never stdout.

### Error Categories and Exit Codes

| Code | Category | When |
|------|----------|------|
| `0`  | Success  | Command completed without error |
| `1`  | Validation error | Invalid field value, missing required argument |
| `2`  | Not found | Task id does not exist in the store |
| `3`  | Usage error | Unknown command, missing required CLI argument |
| `4`  | Unexpected error | Unhandled exception or system-level failure |

### Error Flow by Layer

```
user input
    ↓
cli.js          — catches all errors, sets process.exitCode, prints to stderr
    ↓
taskManager.js  — throws Error on validation failures and not-found cases
    ↓
storage.js      — trusts callers; does not throw (returns null / false on miss)
```

### Layer Responsibilities

#### `bin/task-manager.js`

Wraps the entire CLI invocation in a top-level `try/catch`. Any unhandled error that escapes `cli.js` is caught here, logged with `console.error`, and exits with code `4`.

```js
try {
  await run(process.argv.slice(2));
} catch (err) {
  console.error('Unexpected error:', err.message);
  process.exitCode = 4;
}
```

#### `src/cli.js`

Each command handler wraps its `taskManager.js` call in a `try/catch`. On error, it prints a user-friendly message to stderr and sets the appropriate exit code. Stack traces are suppressed unless `--debug` is passed.

```js
try {
  const task = createTask(args);
  console.log(formatTask(task));
} catch (err) {
  console.error(`Error: ${err.message}`);
  process.exitCode = err.code ?? 1;
}
```

#### `src/taskManager.js`

Throws named errors by attaching a `code` property so `cli.js` can set the correct exit code without string matching.

```js
function notFound(id) {
  const err = new Error(`Task with id ${id} not found`);
  err.code = 2;
  return err;
}

function validationError(message) {
  const err = new Error(message);
  err.code = 1;
  return err;
}
```

#### `src/storage.js`

Does not throw. Returns `null` from `findById` and `false` from `remove` when a task is not present. `taskManager.js` interprets these sentinel values and throws the appropriate error.

### Error Message Conventions

- Messages are lowercase with no trailing period: `'title is required'`
- Include the offending value where helpful: `'Task with id 42 not found'`
- Include the allowed values for enum errors: `'status must be one of: todo, in-progress, done'`
- Never expose internal state (stack frames, variable names) in user-facing messages

### Test Error Handling

Each error path should have a dedicated test using `assert.throws`:

```js
function testCreateTaskThrowsOnMissingTitle() {
  assert.throws(
    () => createTask({ priority: 'high' }),
    (err) => err instanceof Error && /title is required/i.test(err.message)
  );
}

function testGetTaskThrowsOnUnknownId() {
  assert.throws(
    () => getTask(999),
    (err) => err instanceof Error && err.code === 2
  );
}
```
