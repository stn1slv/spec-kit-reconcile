# Implementation Plan: Notifications

**Branch**: `002-notifications`
**Date**: 2026-08-02
**Spec**: [spec.md](./spec.md)

## Summary

A scheduled reminder worker beside the existing API, sharing its database.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, APScheduler 3.10
**Storage**: PostgreSQL 16
**Project Type**: web-service with background worker
**Constraints**: single region
**Scale/Scope**: 5000 notifications per day

## Project Structure

```
src/
├── api/
├── notifications/   # worker, templates, channels
└── components/
```

## Routing & Navigation

- `GET /api/v1/notifications` — list the caller's notifications

## Integration Contracts

- The worker reads tasks through the internal task service; no new public contract.

## Configuration

- `REMINDER_LEAD_HOURS` — how far ahead a reminder fires, default 24.

## Testing Strategy

- Integration test for `GET /api/v1/notifications`.
- Unit tests for the digest grouping rule.

## Constitution Check

No violations. Notifications reference tasks by id and add no ownership semantics.
