# Feature Specification: Task Manager

**Feature Branch**: `001-task-manager`
**Created**: 2026-07-28
**Status**: Completed
**Input**: Team needs a shared place to track work with a clear owner per task.

## User Scenarios & Testing

### User Story 1 - Create and assign a task (Priority: P1)

A team member creates a task and assigns it to exactly one owner, who then sees it in their list.

**Why this priority**: Nothing else in the product works until a task can exist and belong to someone.

**Independent Test**: Create a task, assign it, and confirm it appears for the assignee and nobody else.

**Acceptance Scenarios**:

1. **Given** an authenticated member, **When** they create a task and pick an owner, **Then** the task appears in that owner's list with status `open`.
2. **Given** a task with an owner, **When** a second member is picked as owner, **Then** the previous owner is replaced rather than added.

### User Story 2 - Archive completed work (Priority: P2)

An owner archives a finished task so it leaves the active list without being destroyed.

**Why this priority**: The active list becomes unusable within a sprint if finished work cannot leave it.

**Independent Test**: Archive a task and confirm it leaves the active list and is still retrievable.

**Acceptance Scenarios**:

1. **Given** a task marked done, **When** its owner archives it, **Then** it leaves the active list and is retrievable from the archive.

### Edge Cases

- A member is deactivated while still owning open tasks.
- Two members archive the same task within the same second.
- A task is archived and then reopened before the retention sweep runs.

## Requirements

- **FR-001**: Users MUST be able to create a task with a title, a description, and a due date.
- **FR-002**: Users MUST be able to assign a task to exactly one owner.
- **FR-003**: The system MUST show each member the tasks they own, ordered by due date.
- **FR-004**: Users MUST be able to archive a task they own.
- **FR-005**: Archived tasks MUST be deleted 30 days after archival.

### Key Entities

- **Task**: id, title, description, due_date, owner_id, status, archived_at
- **User**: id, display_name, email, active

## Success Criteria

- **SC-001**: A member can create and assign a task in under 30 seconds.
- **SC-002**: The owner's task list renders in under 300 ms for 500 tasks.

## Assumptions

- Every member authenticates through the company SSO provider.
- A task belongs to exactly one team; cross-team tasks are out of scope.
