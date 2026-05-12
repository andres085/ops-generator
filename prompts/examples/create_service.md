Use the process defined in @prompts/create_team.md.
Complete the task below.

# Task Overview

Create `notification-service-be-go`, a new standalone Go microservice that listens for domain events on NATS and sends notifications via email (SendGrid) and webhook.

The service is event-driven — it does not expose a public API. It subscribes to a NATS subject, processes each event, looks up notification subscriptions from its own database, and delivers notifications through the configured channel.

- **Artifact**: `notification-service`
- **Artifact type**: Go microservice
- **Stack**: Go / NATS JetStream / PostgreSQL / SendGrid
- **Reference codebase**: `../user-service-be-go`
- **Reference implementation**: `../user-service-be-go/internal/handler/` (NATS handler pattern)

# Events Consumed

## `domain.events.v1` (NATS subject)

The service consumes all messages published to this subject. Each message is a JSON payload:

```json
{
  "event_id": "string (UUID)",
  "event_type": "string (e.g., user.created, order.completed)",
  "organization_id": "string",
  "entity_id": "string",
  "payload": "object (event-specific data)",
  "occurred_at": "string (ISO 8601)"
}
```

Processing guarantee: at-least-once delivery. The service must be idempotent — processing the same `event_id` twice must not send duplicate notifications.

# Notification Subscriptions

Subscriptions are stored in the service's own PostgreSQL database:

| Column | Type | Description |
|--------|------|-------------|
| `subscription_id` | `uuid` | Primary key |
| `organization_id` | `varchar` | Organization that owns the subscription |
| `event_type` | `varchar` | Event type to match (supports `*` wildcard) |
| `channel` | `varchar` | `email` or `webhook` |
| `destination` | `varchar` | Email address or webhook URL |
| `template_id` | `varchar` | SendGrid template ID (email only, nullable) |
| `active` | `bool` | Whether the subscription is active |
| `created_at` | `timestamptz` | |

# Delivery Channels

## Email (SendGrid)

- Use SendGrid Dynamic Templates if `template_id` is set; otherwise use a generic template
- Template variables: `event_type`, `entity_id`, `occurred_at`, and all fields from `payload`
- Max 3 delivery attempts with exponential backoff (base 1s)

## Webhook

- POST the full event payload as JSON to the `destination` URL
- Include `X-Event-ID` and `X-Event-Type` headers
- Timeout: 10s per attempt
- Max 3 delivery attempts with exponential backoff (base 2s)
- Accept any 2xx response as success

# Idempotency

Track delivered notifications in a `notification_log` table:

| Column | Type | Description |
|--------|------|-------------|
| `log_id` | `uuid` | Primary key |
| `event_id` | `uuid` | The source event |
| `subscription_id` | `uuid` | The matched subscription |
| `status` | `varchar` | `delivered`, `failed` |
| `attempts` | `int` | Number of delivery attempts |
| `delivered_at` | `timestamptz` | |
| `error` | `text` | Last error (nullable) |

Before delivering, check if `(event_id, subscription_id)` already exists in `notification_log`. If it does, skip delivery and ACK the message.

# Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | yes | — | PostgreSQL connection string |
| `NATS_URL` | yes | — | NATS server URL |
| `NATS_SUBJECT` | no | `domain.events.v1` | Subject to subscribe to |
| `NATS_CONSUMER` | no | `notification-service` | Consumer name |
| `SENDGRID_API_KEY` | yes | — | SendGrid API key |
| `SENDGRID_FROM_EMAIL` | yes | — | Sender email address |
| `WEBHOOK_TIMEOUT_SEC` | no | `10` | Webhook delivery timeout |
| `MAX_DELIVERY_ATTEMPTS` | no | `3` | Max retries per channel |
| `LOG_LEVEL` | no | `info` | Log level |
| `PORT` | no | `8080` | Health check HTTP port |

# Health & Observability

Expose HTTP endpoints (follow the reference codebase pattern):
- `GET /health` — always 200, no dependencies
- `GET /ready` — 200 if DB and NATS are connected, 503 otherwise
- `GET /metrics` — Prometheus metrics

Key metrics to expose:
- `notifications_processed_total` (labels: `event_type`, `channel`, `status`)
- `notifications_delivery_duration_seconds` (histogram, labels: `channel`)
- `subscription_match_total` (labels: `event_type`, `matched`)

# Error Handling Strategy

| Error | Action |
|-------|--------|
| DB unavailable at startup | Fail fast, exit with error |
| NATS unavailable at startup | Fail fast, exit with error |
| Subscription lookup fails | NAK message (retry) |
| No matching subscriptions | ACK message, log at debug level |
| Delivery fails after max attempts | Mark `notification_log` as `failed`, ACK message |
| Idempotency check fails | ACK message, skip |
| Invalid event payload | Log warning, ACK message (don't retry bad data) |

# Test Infrastructure

The `test-writer` should provide:
- PostgreSQL container seeded with test subscriptions
- NATS JetStream container
- Stub SendGrid server (HTTP mock that records calls)
- Stub webhook server (HTTP server that records received webhooks)

Test scenarios:
1. Event with matching email subscription → SendGrid stub receives the call
2. Event with matching webhook subscription → webhook stub receives POST with correct headers
3. Duplicate `event_id` → only one notification delivered (idempotency)
4. Wildcard subscription (`event_type = *`) → matches any event type
5. Inactive subscription → no notification sent
6. SendGrid failure → retry up to `MAX_DELIVERY_ATTEMPTS`, mark as `failed`
7. No matching subscriptions → event ACKed, no notification

# References

- Reference codebase: `../user-service-be-go` — follow all patterns from this service
- NATS JetStream Go client: https://github.com/nats-io/nats.go
- SendGrid Go client: https://github.com/sendgrid/sendgrid-go
- SendGrid Dynamic Templates: https://docs.sendgrid.com/ui/sending-email/how-to-send-an-email-with-dynamic-templates
