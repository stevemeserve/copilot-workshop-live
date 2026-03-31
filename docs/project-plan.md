# Task Manager CLI Application - Project Plan

## Overview

Build a Node.js-based CLI task manager with in-memory storage, supporting full CRUD operations on tasks with properties (title, description, status, priority, timestamps). The application will implement core commands (add, list, update, delete, filter, sort) with a modular architecture separating CLI handling, business logic, and data storage.

## Task Properties

Each task contains:
- **id**: Unique identifier (auto-generated)
- **title**: Task name (required)
- **description**: Task details (optional)
- **status**: One of `["todo", "in-progress", "done"]` (default: "todo")
- **priority**: One of `["low", "medium", "high"]` (default: "medium")
- **createdAt**: ISO timestamp (auto-generated)
- **updatedAt**: ISO timestamp (auto-generated, updated on modifications)

## Core Features

### CRUD Operations
- **Create**: Add new tasks with title, description, and optional priority
- **Read**: Retrieve and display tasks
- **Update**: Modify task properties (status, priority, description)
- **Delete**: Remove tasks by id

### Filtering
- Filter tasks by status (todo, in-progress, done)
- Filter tasks by priority (low, medium, high)
- Combine multiple filters

### Sorting
- Sort by priority (high → medium → low)
- Sort by creation date (newest first or oldest first)

### Display
- List all tasks with formatted output (table or clean text)
- Show task details with all properties
- Help/usage documentation

## Project Structure

```
task-manager/
├── src/
│   ├── cli.js              # Command parsing and routing
│   ├── taskManager.js      # Business logic layer
│   ├── storage.js          # In-memory data store
│   ├── constants.js        # Task statuses, priorities, validation rules
│   └── utils.js            # Helper functions (formatting, validation)
├── bin/
│   └── task-manager.js     # Main executable entry point
├── package.json            # Project metadata
├── README.md               # User documentation
└── docs/
    └── project-plan.md     # This file
```

## Implementation Phases

### Phase 1: Project Setup & Architecture (Preparatory)
1. Define project structure with separate modules for CLI, business logic, and storage
2. Set up command parsing framework (Node.js built-ins or minimist)
3. Define Task model/schema with required properties
4. Create `constants.js` with valid task statuses and priorities

### Phase 2: Core Data Management
5. Implement `storage.js` module with in-memory data structure
   - Task CRUD operations (create, read, update, delete)
   - Helper methods for accessing tasks by id or filtering
6. Implement `taskManager.js` business logic layer
   - Wraps storage operations with validation
   - Handles task creation with auto-generated id and timestamps
   - Provides filtering by status/priority
   - Provides sorting by priority or createdAt

### Phase 3: CLI Commands (can be parallelized)
7. Implement `add` command — create new task with title, description, priority
8. Implement `list` command — display all tasks with optional filtering/sorting
9. Implement `update` command — modify task properties by id
10. Implement `delete` command — remove task by id
11. Implement `filter` command — query tasks by status or priority
12. Implement `--help` command and documentation

### Phase 4: Polish & Testing
13. Implement formatted output (tables or clean text display)
14. Add input validation (validate priority values, status values, required fields)
15. Add error handling with user-friendly error messages
16. Manual testing of all commands and edge cases

## Files to Create

- `src/cli.js` — Entry point, command parsing and routing
- `src/taskManager.js` — Business logic (validation, filtering, sorting)
- `src/storage.js` — In-memory data store and CRUD operations
- `src/constants.js` — Task statuses, priorities, validation rules
- `src/utils.js` — Helper functions (formatting, validation)
- `bin/task-manager.js` — Main executable entry point
- `package.json` — Project metadata and dependencies

## Error Handling Conventions

### Error Categories

#### 1. **Validation Errors** (Input validation failures)
- **Exit Code**: 1
- **Message Format**: `Error: [field] must be [requirement]. Received: [value]`
- **Examples**:
  - `Error: title must be a non-empty string. Received: ""`
  - `Error: priority must be one of: low, medium, high. Received: "urgent"`
  - `Error: status must be one of: todo, in-progress, done. Received: "completed"`

#### 2. **Resource Not Found Errors**
- **Exit Code**: 2
- **Message Format**: `Error: Task with id [id] not found`
- **Context**: Triggered by update or delete operations on non-existent task ids
- **Example**: `Error: Task with id 99 not found`

#### 3. **Command Usage Errors**
- **Exit Code**: 3
- **Message Format**: `Error: Invalid command. Use 'task-manager --help' for usage`
- **Context**: Unknown commands, incorrect command syntax
- **Example**: `Error: Unknown command 'remove'. Did you mean 'delete'?`

#### 4. **System/Runtime Errors**
- **Exit Code**: 4
- **Message Format**: `Error: [description]`
- **Context**: Unexpected conditions, internal failures
- **Do Not Expose**: Stack traces or internal implementation details

### Error Handling Standards

- **User-Facing Messages**: Keep brief, actionable, and non-technical
- **Logging**: Log full error details internally (when logging is implemented)
- **Graceful Degradation**: Fail safely without corrupting in-memory state
- **No Partial Operations**: Rollback incomplete operations to maintain data consistency
- **Consistent Formatting**: All error messages follow the pattern: `Error: [description]`

## Input Validation Rules

### Task Fields

#### Title
- **Type**: String
- **Required**: Yes
- **Validation Rules**:
  - Must not be empty or whitespace-only
  - Must be between 1-200 characters
  - Trim leading and trailing whitespace
- **Error If Invalid**: `Error: title must be a non-empty string between 1-200 characters. Received: "[value]"`

#### Description
- **Type**: String
- **Required**: No
- **Validation Rules**:
  - If provided, must be between 0-1000 characters
  - Trim leading and trailing whitespace
  - Allow empty string (treated as no description)
- **Error If Invalid**: `Error: description must be between 0-1000 characters. Received length: [length]`

#### Status
- **Type**: String (enum)
- **Required**: No (default: "todo")
- **Valid Values**: `["todo", "in-progress", "done"]`
- **Validation Rules**:
  - Must match one of valid values (case-insensitive)
  - Convert to lowercase for storage
- **Error If Invalid**: `Error: status must be one of: todo, in-progress, done. Received: "[value]"`

#### Priority
- **Type**: String (enum)
- **Required**: No (default: "medium")
- **Valid Values**: `["low", "medium", "high"]`
- **Validation Rules**:
  - Must match one of valid values (case-insensitive)
  - Convert to lowercase for storage
- **Error If Invalid**: `Error: priority must be one of: low, medium, high. Received: "[value]"`

#### Task ID
- **Type**: Integer
- **Generation**: Auto-incremented, starting from 1
- **Validation Rules**:
  - Must be a positive integer
  - Must be unique
  - When provided in commands (update, delete), must reference existing task
- **Error If Invalid**: `Error: Invalid task id. Expected positive integer, received: "[value]"`

### Command-Level Validation

#### `add` Command
- **Required Arguments**: title
- **Optional Arguments**: description, priority
- **Validation**:
  - Validate title according to title rules above
  - Validate priority if provided
  - Set defaults: status="todo", priority="medium"
- **Error**: Exit with code 1

#### `update` Command
- **Required Arguments**: task-id
- **Optional Arguments**: status, priority, description, title
- **Validation**:
  - Validate task-id exists
  - Validate each provided field according to field-specific rules
  - Allow partial updates (only update provided fields)
- **Error**: Exit with code 1 or 2 (depending on error type)

#### `delete` Command
- **Required Arguments**: task-id
- **Validation**:
  - Validate task-id exists
  - Confirm operation (optional, for safety)
- **Error**: Exit with code 2

#### `list` Command
- **Optional Arguments**: status, priority, sort
- **Validation**:
  - If status provided, must be valid status value
  - If priority provided, must be valid priority value
  - If sort provided, must be one of: "priority", "date"
- **Error**: Exit with code 1

### Validation Implementation

**Location**: `src/utils.js` - Create dedicated validator functions
- `validateTitle(value)` → throws or returns normalized value
- `validateDescription(value)` → throws or returns normalized value
- `validateStatus(value)` → throws or returns normalized value
- `validatePriority(value)` → throws or returns normalized value
- `validateTaskId(value)` → throws or returns parsed integer
- `validateTask(taskObject)` → validates complete task object

**Location**: `src/taskManager.js` - Call validators before operations
- All public methods validate inputs before proceeding
- Return operation results or throw descriptive errors

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| **In-memory storage** | Simplifies initial implementation; data resets on app restart |
| **Modular architecture** | Separates concerns: CLI, business logic, data storage |
| **No external dependencies** | Use Node.js built-ins where possible (minimist only if needed) |
| **Task ID generation** | Incrementing counter for simplicity |
| **Status values** | `["todo", "in-progress", "done"]` — clear and intuitive |
| **Priority values** | `["low", "medium", "high"]` — standard three-level system |
| **Exit codes** | Standardized codes (1: validation, 2: not found, 3: usage, 4: system error) |
| **Case-insensitive enums** | User convenience; normalized to lowercase in storage |

## CLI Command Examples

```bash
# Create a task
task-manager add "Buy groceries" --description "Milk, eggs, bread" --priority high

# List all tasks
task-manager list

# List tasks with filtering
task-manager list --status todo
task-manager list --priority high

# List tasks sorted
task-manager list --sort priority
task-manager list --sort date

# Update a task
task-manager update 1 --status in-progress
task-manager update 2 --priority medium

# Delete a task
task-manager delete 1

# Display help
task-manager --help
```

## Acceptance Criteria

### Core Functionality
- [ ] Create a task with title, description, and priority
- [ ] View all tasks in a formatted list
- [ ] Update a task's status or priority
- [ ] Delete a task by id
- [ ] Filter tasks by status
- [ ] Filter tasks by priority
- [ ] Sort tasks by priority
- [ ] Sort tasks by creation date

### Data Integrity
- [ ] Tasks persist in memory during session
- [ ] Auto-generated ids are unique
- [ ] Timestamps are accurate (createdAt and updatedAt)
- [ ] Invalid priority/status values are rejected
- [ ] Required fields (title) are validated

### Error Handling
- [ ] Validation errors return exit code 1 with clear messages
- [ ] Not found errors return exit code 2 with context
- [ ] Usage errors return exit code 3 with help suggestion
- [ ] Error messages are user-friendly and actionable

### User Experience
- [ ] Formatted, readable output (table or clean text)
- [ ] Clear error messages for invalid input
- [ ] Help documentation is accessible
- [ ] Commands are intuitive and consistent

## Testing Strategy

1. **Unit Tests** (for each module)
   - Create task with required properties
   - Update task properties correctly
   - Delete task removes it from storage
   - Filter operations return correct subsets
   - Sort operations order tasks correctly
   - Validation functions reject invalid inputs appropriately

2. **Integration Tests**
   - CLI commands parse arguments correctly
   - Commands invoke business logic correctly
   - Output formatting is consistent
   - Error codes and messages are correct

3. **Manual Testing Scenarios**
   - Create multiple tasks with varying properties
   - Filter by status and verify results
   - Sort by priority and verify ordering
   - Update an existing task and confirm changes
   - Delete a task and confirm removal
   - Test edge cases: empty list, invalid inputs, boundary conditions
   - Test all error conditions with expected exit codes

## Future Enhancements

- Persistent storage (JSON file or database)
- Recurring tasks
- Task categories/projects
- Task dependencies
- Time tracking/estimates
- Subtasks/checklist items
- Export functionality (CSV, JSON, calendar)
- Configuration file for preferences
- Undo/redo history
- Shell aliases and tab completion
