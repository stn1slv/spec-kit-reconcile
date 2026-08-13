# Implementation Plan: Task Manager

**Branch**: `001-task-manager`
**Date**: 2026-07-28
**Spec**: [spec.md](./spec.md)

## Summary

A single-service web application: a React front end over a Python API, with tasks owned by exactly one user.

## Technical Context

**Language/Version**: Python 3.12, TypeScript 5.4
**Primary Dependencies**: FastAPI 0.115, React 18.3
**Storage**: PostgreSQL 16
**Project Type**: web-service
**Constraints**: single region
**Scale/Scope**: 200 concurrent users

## Project Structure

```
src/
├── api/            # FastAPI routers
├── components/     # React components
└── router/         # client-side routes
tests/
└── integration/
```

## Routing & Navigation

- `GET /api/v1/tasks` — list the caller's tasks
- `POST /api/v1/tasks` — create a task
- `POST /api/v1/tasks/{id}/archive` — archive a task
- Client route `/tasks` renders the task list; it is reachable from the sidebar.

## Integration Contracts

- Task payloads carry `owner_id`; the API rejects a payload naming more than one owner.

## Configuration

- `RETENTION_SWEEP_CRON` — schedule for the nightly archival sweep.

## Testing Strategy

- Integration tests for the three task routes live in `tests/integration/`.
- The nightly retention sweep is covered by a scheduled-job test.

## Constitution Check

No violations. Every task carries exactly one `owner_id`, and FR-005 states the retention rule.
