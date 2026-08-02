---
name: Collect a cross-border payment with a Xflow receivable
description: Create an account, raise a receivable for an export invoice, confirm
  it, and reconcile incoming deposits so funds settle as an INR payout.
api: openapi/xflow-openapi-original.yml
operations:
- CreateAccount
- ActivateAccount
- CreateReceivable
- ConfirmReceivable
- ListDeposits
- ReconcileReceivable
- ListPayouts
---

# Collect a cross-border payment with a Xflow receivable

Authenticate every request with `Authorization: Bearer sk_test_...` (testmode) or
`sk_live_...` (live). Platforms acting for a connected user must also send the
`Xflow-Account: <account_id>` header.

1. **Create the account** — `CreateAccount` (`POST /v1/accounts`) for the Indian
   business receiving funds. Add required persons via `CreatePerson` and a payout
   address, then `ActivateAccount` (`POST /v1/accounts/{account}/activate`). In
   testmode, dummy data activates instantly.
2. **Raise the receivable** — `CreateReceivable` (`POST /v1/receivables`) with the
   invoice amount, currency, purpose_code, and invoice details. Watch for
   `receivable_*` error codes (e.g. `receivable_invoice_reference_number_exists`).
3. **Confirm it** — `ConfirmReceivable` (`POST /v1/receivables/{receivable}/confirm`).
   In testmode the receivable auto-transitions to `active`.
4. **Track incoming funds** — the buyer pays into the account's local collection
   address; watch `deposit.status.completed` webhooks or poll `ListDeposits`
   (`GET /v1/deposits`).
5. **Reconcile** — `ReconcileReceivable`
   (`POST /v1/receivables/{receivable}/reconcile`) to lock deposits against the
   receivable. Respect `receivable_reconcile_amount_insufficient` /
   `receivable_reconcile_amount_exceed` errors.
6. **Settlement** — payouts settle to the Indian bank account; track via
   `ListPayouts` (`GET /v1/payouts`) and the `payout.status.settled` event, which
   carries the bank UTR (`unique_transaction_reference`).

Conventions: cursor pagination (`limit` 1-10, `starting_after`, `ending_before`,
`has_next`); errors arrive as `{ object: "error", http_status_code, errors: [{code,
message}] }`; there is **no idempotency-key support**, so never blind-retry POSTs —
re-list to verify state first.
