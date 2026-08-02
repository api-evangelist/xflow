---
name: Invoice a buyer with a Xflow payment link
description: Turn confirmed receivables into a hosted payment link the
  international buyer can pay, then manage its lifecycle.
api: openapi/xflow-openapi-original.yml
operations:
- CreateReceivable
- ConfirmReceivable
- CreatePaymentLink
- RetrievePaymentLink
- ExtendPaymentLink
- ExpirePaymentLink
---

# Invoice a buyer with a Xflow payment link

Authenticate with `Authorization: Bearer sk_...`; platforms add `Xflow-Account`.

1. **Prepare receivables** — `CreateReceivable` then `ConfirmReceivable` for each
   invoice to collect.
2. **Create the link** — `CreatePaymentLink` (`POST /v1/payment_links`) referencing
   the receivable ids. Note `payment_link_receivable_status_invalid` (receivables
   must be in a linkable status) and
   `payment_link_non_us_account_non_usd_currency` constraints.
3. **Share and monitor** — send the returned `link` URL to the buyer;
   `RetrievePaymentLink` (`GET /v1/payment_links/{payment_link}`) reports status
   and `expires_at`.
4. **Manage lifecycle** — `ExtendPaymentLink` to push out expiry (see
   `payment_link_extend_invalid`), `ExpirePaymentLink` to kill a link early,
   `ActivatePaymentLink`/`DeactivatePaymentLink` to toggle availability.
5. **After payment** — deposits arrive and reconcile against the receivables;
   watch `deposit.status.completed` and `receivable.amount_reconciled.updated`
   webhook events.

No idempotency keys exist — check for an existing active link with
`ListPaymentLinks` before creating a duplicate.
