# Implementation Plan: Attachments

**Branch**: `003-attachments`
**Date**: 2026-08-06
**Spec**: [spec.md](./spec.md)

## Summary

File attachments on tasks, stored in the object store and addressed by content hash.

## Technical Context

**Language/Version**: Python 3.12, TypeScript 5.4
**Primary Dependencies**: FastAPI 0.115, boto3 1.35
**Storage**: PostgreSQL 16, S3-compatible object store
**Project Type**: web-service
**Constraints**: single region
**Scale/Scope**: 25 MB per file, 200 uploads per hour

## Project Structure

```
src/
├── api/
├── attachments/
└── components/
```

## Routing & Navigation

- `POST /api/v1/tasks/{id}/attachments` — upload
- `GET /api/v1/attachments/{id}` — download

## Integration Contracts

- Upload is a multipart POST; the response carries the attachment id and content hash.

## Configuration

- `OBJECT_STORE_BUCKET`, `MAX_ATTACHMENT_BYTES`.

## Testing Strategy

- Integration tests for the upload and download routes.

## Constitution Check

No violations.

### Revision: Implementation Sync [2026-08-11] [Sync: upload-progress]
- Reason: The progress indicator shipped without a cancel affordance, and the plan described neither.
- Items: Routing & Navigation → "POST /api/v1/tasks/{id}/attachments"
