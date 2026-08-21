# Feature Specification: Notifications

**Feature Branch**: `002-notifications`
**Created**: 2026-08-02
**Status**: In Progress

## What We're Building

Deadline reminders for task owners, delivered by email and in-app.

### Flow 1 - Deadline reminder (Priority: P1)

A task owner is reminded before a task is due, so nothing slips silently.

**Why this priority**: Reminders are the reason the team asked for this feature at all.

**Checks**:

1. **Given** a task due in 24 hours, **When** the reminder job runs, **Then** its owner receives exactly one reminder.
2. **Given** a task with no due date, **When** the reminder job runs, **Then** no reminder is sent.

### Flow 2 - Digest (Priority: P2)

An owner with several due tasks receives one digest rather than one message per task.

**Checks**:

1. **Given** three tasks due the same day, **When** the digest runs, **Then** the owner receives one message listing all three.

### Things That Can Go Wrong

- The mail provider rejects a recipient address that was valid at task creation.
- A reminder fires for a task archived in the same minute.

## Behaviour Rules

- **REQ-001**: The system MUST send a task's owner a reminder 24 hours before its due date.
- **REQ-002**: The system MUST send at most one digest per owner per day.
- **REQ-003**: Owners MUST be able to turn reminders off entirely. **RETIRED** — a blanket opt-out was dropped after the design review.

### Things We Store

- **Notification**: id, recipient_id, task_id, channel, sent_at, delivery_status

## How We'll Know It Works

- **M1**: 99% of reminders are delivered within 60 seconds of the scheduled time.
- **M2**: No owner receives more than one digest in a calendar day.

## Open Questions

- Whether a reminder should fire for a task whose due date is changed after the reminder was queued.
