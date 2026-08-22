# Feature Specification: Attachments

**Feature Branch**: `003-attachments`
**Created**: 2026-08-06
**Status**: In Progress

## User Scenarios & Testing

### User Story 1 - Attach a file to a task (Priority: P1)

An owner attaches a file to a task so the context lives with the work.

**Why this priority**: Every pilot team asked for it before anything else in this feature.

**Independent Test**: Attach a file to a task, reload, and confirm it is listed and downloadable.

**Acceptance Scenarios**:

1. **Given** a task the caller owns, **When** they upload a file under the size limit, **Then** it is listed on the task with its name and size.
2. **Given** an upload in progress, **When** the caller navigates away, **Then** the upload is cancelled rather than orphaned.

### Edge Cases

- An upload completes after its task has been archived.
- ~~A file whose name contains a path separator is rejected outright.~~
- The storage backend returns a 503 midway through a multipart upload.

## Requirements

- **FR-001**: Users MUST be able to attach files up to 25 MB to a task they own.
- **FR-002**: ~~Attachments MUST be stored on the application server's local disk.~~ Attachments MUST be stored in the object store, addressed by content hash.
  **Bugfix**: 2026-08-10 — [BUG-001] Local disk storage lost attachments on every deploy; moved to the object store.
- **FR-003**: The system MUST show upload progress while a file is transferring, and MUST offer a cancel affordance while it does. **SUPERSEDED by FR-005** (refined to cover the transfer outcome).
- **FR-004**: Users MUST be able to delete an attachment from a task they own.
- **FR-005**: While a file is transferring, the system MUST display its progress and MUST allow the user to cancel it, and MUST report the outcome once the transfer ends.

### Key Entities

- **Attachment**: id, task_id, filename, size_bytes, content_hash, uploaded_at

## Success Criteria

- **SC-001**: A 10 MB upload completes in under 5 seconds on a 50 Mbps connection.

## Assumptions

- Files are scanned for malware by the storage layer, not by this feature.

### Revision: Implementation Sync [2026-08-11] [Sync: upload-progress]
- Reason: The progress indicator shipped without a cancel affordance, and the spec described neither.
- Items: FR-003, User Story 1 → "Given an upload in progress"
