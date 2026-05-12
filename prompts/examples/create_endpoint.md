Use the process defined in @prompts/create_team.md.
Complete the task below.

# Task Overview

Add a `POST /reports` endpoint to the `analytics-api` NestJS service.

The endpoint triggers the async generation of a usage report for a given organization. It enqueues a background job and immediately returns a job ID. A separate worker processes the job and stores the result in S3.

- **Artifact**: `reports-endpoint`
- **Artifact type**: REST API endpoint + background job handler
- **Stack**: TypeScript / NestJS / BullMQ / S3
- **Reference codebase**: `../analytics-api`
- **Reference implementation**: `../analytics-api/src/exports/exports.controller.ts` (similar async pattern)

# API Contract

## POST /reports

**Request**
```json
{
  "organization_id": "string (required)",
  "date_from": "string (ISO 8601, required)",
  "date_to": "string (ISO 8601, required)",
  "format": "csv | json (optional, default: csv)"
}
```

**Response 202 Accepted**
```json
{
  "job_id": "string",
  "organization_id": "string",
  "status": "queued",
  "created_at": "string (ISO 8601)"
}
```

**Errors**

| Status | Condition |
|--------|-----------|
| 400 | Missing required fields or invalid date format |
| 422 | `date_to` is before `date_from` |
| 500 | Failed to enqueue job |

## GET /reports/:job_id

**Response 200**
```json
{
  "job_id": "string",
  "organization_id": "string",
  "status": "queued | processing | completed | failed",
  "result_url": "string (S3 pre-signed URL, only when status=completed)",
  "error": "string (only when status=failed)",
  "created_at": "string (ISO 8601)",
  "completed_at": "string (ISO 8601, only when status=completed or failed)"
}
```

| Status | Condition |
|--------|-----------|
| 404 | Job not found |

# Background Job

The BullMQ worker processes `generate-report` jobs:

1. Fetch organization usage data from the `events` table for the requested date range
2. Aggregate by day and event type
3. Format as CSV or JSON depending on `format` field
4. Upload to S3 at `reports/{organization_id}/{job_id}.{ext}`
5. Generate a pre-signed URL valid for 24 hours
6. Update job status to `completed` with the pre-signed URL

# Data Model

Persist job state in PostgreSQL table `report_jobs`:

| Column | Type | Description |
|--------|------|-------------|
| `job_id` | `uuid` | Primary key, generated on creation |
| `organization_id` | `varchar` | Organization that requested the report |
| `date_from` | `date` | Start of report range |
| `date_to` | `date` | End of report range |
| `format` | `varchar` | `csv` or `json` |
| `status` | `varchar` | `queued`, `processing`, `completed`, `failed` |
| `result_url` | `varchar` | S3 pre-signed URL (nullable) |
| `error` | `text` | Error message (nullable) |
| `created_at` | `timestamptz` | Creation timestamp |
| `completed_at` | `timestamptz` | Completion timestamp (nullable) |

# Error Handling Strategy

| Error | Action | HTTP Status |
|-------|--------|-------------|
| Validation failure | Return 400/422 immediately, do not enqueue | 400/422 |
| Queue unavailable | Log error, return 500 | 500 |
| DB unavailable during GET | Return 503 | 503 |
| Data fetch fails in worker | Mark job as `failed`, store error | — |
| S3 upload fails in worker | Retry 3x with backoff; mark `failed` if all fail | — |
| Pre-sign URL fails | Mark job as `failed`, store error | — |

**Principle**: HTTP errors are immediate (sync path). Worker errors update job status to `failed` with an error message — they never throw unhandled exceptions.

# Test Data

The `test-writer` should spin up:
- PostgreSQL container with the `report_jobs` table (migration from `../analytics-api/db/migrations/`)
- Redis container for BullMQ
- Mocked S3 using LocalStack

Seed data: at least one organization with 30 days of events across 5 event types.

# References

- Reference implementation: `../analytics-api/src/exports/` — same async pattern (enqueue → poll)
- BullMQ NestJS integration: https://docs.nestjs.com/techniques/queues
- AWS SDK v3 S3 pre-signed URLs: https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/modules/_aws_sdk_s3_request_presigner.html
