---
name: Monitor Xflow money movement with webhooks
description: Register a signed webhook endpoint, subscribe to account, receivable,
  deposit, and payout events, and reconcile against the events API.
api: openapi/xflow-openapi-original.yml
operations:
- CreateWebhookEndpoint
- ActivateWebhookEndpoint
- ListEvents
- RetrieveEvent
- DeactivateWebhookEndpoint
---

# Monitor Xflow money movement with webhooks

1. **Register** — `CreateWebhookEndpoint` (`POST /v1/webhook_endpoints`) with your
   HTTPS URL and the `enabled_events` you need (e.g. `deposit.status.completed`,
   `receivable.status.completed`, `payout.status.settled`, `payout.status.failed`).
   Store the returned endpoint `secret`.
2. **Activate** — `ActivateWebhookEndpoint`
   (`POST /v1/webhook_endpoints/{webhook_endpoint}/activate`).
3. **Verify every delivery** — reconstruct the signed content, compute the
   expected signature with the endpoint secret, and check the timestamp before
   trusting a payload. Only accept deliveries from Xflow's documented source IPs.
4. **Handle duplicates** — deliveries can repeat; deduplicate on event `id`.
   Retries are automatic on failure, so return 2xx quickly.
5. **Reconcile** — cross-check with `ListEvents` (`GET /v1/events`, filterable by
   type) and `RetrieveEvent` for the authoritative record; each event links to its
   object via `linked_object` + `linked_id`.
6. **Rotate/retire** — `DeactivateWebhookEndpoint` before decommissioning an
   endpoint. Note `webhook_endpoint_limit_exceeded` caps how many endpoints an
   account may register.
