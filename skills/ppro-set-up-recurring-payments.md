---
name: ppro-set-up-recurring-payments
description: Establish a consumer mandate as a PPRO payment agreement and charge against it on a schedule, including the stored instrument and revocation.
api: PPRO Global API
base_url: https://api.eu.ppro.com
operations:
  - createAgreement
  - confirm
  - fetchPaymentAgreement
  - createCharge
  - revokeAgreement
  - createPaymentInstrument
  - fetchPaymentInstrument
  - deletePaymentInstrument
generated: '2026-08-26'
method: generated
source: openapi/ppro-payment-agreements-openapi.yml, openapi/ppro-payment-instruments-openapi.yml, https://developerhub.ppro.com/global-api/docs/recurring-payments
---

# Set up recurring payments with PPRO

Verified against `openapi/ppro-payment-agreements-openapi.yml` and `openapi/ppro-payment-instruments-openapi.yml`.

## Step 1 — create the agreement

`createAgreement` — `POST /v1/payment-agreements`

The agreement is the consumer's mandate. Send the `amount` with an `amountType` of `MAX`, `EXACT` or `VARIABLE` (use `amountType`, not the deprecated `amount.type`), the `frequency`, the `consumer`, the instrument details for the method, and `authenticationSettings` with your `returnUrl`.

For SEPA Direct Debit the bank-account instrument carries a `debitMandateId`. For UPI use the UPI AutoPay instrument shape. Not every payment method supports recurring — check the method page first.

## Step 2 — complete authentication

If more data is required, `confirm` — `POST /v1/payment-agreements/{agreementId}/authorizations`.

## Step 3 — wait for ACTIVE

Do not charge until the agreement is `ACTIVE`. Listen for `PAYMENT_AGREEMENT_ACTIVE`; `PAYMENT_AGREEMENT_FAILED` means setup did not complete. `fetchPaymentAgreement` — `GET /v1/payment-agreements/{agreement-id}` — confirms state.

Charging an inactive agreement produces failure code `AGREEMENT_INACTIVE`.

## Step 4 — charge on the schedule

`createCharge` — `POST /v1/payment-agreements/{agreement-id}/payment-charges`

Each cycle is a new charge with its own idempotency key. Never reuse a key across billing periods — reuse would replay the previous cycle's response instead of taking a new payment.

For card agreements, send the Network Transaction Identifier (and, on Mastercard, the TLID returned on the original consumer-initiated authorization) so the scheme links the chain of merchant-initiated transactions.

The resulting charge behaves exactly like a one-off charge: capture, refund and void all apply.

## Step 5 — stop cleanly

`revokeAgreement` — `POST /v1/payment-agreements/{agreement-id}/revocations`

Revocation is terminal. Stop scheduling immediately after it succeeds. Consumers and providers can also revoke — handle `PAYMENT_AGREEMENT_REVOKED_BY_CONSUMER`, `..._BY_MERCHANT` and `..._BY_PROVIDER`, and stop billing on any of them. Charging a revoked agreement returns `AGREEMENT_REVOKED`.

## Stored instruments

`createPaymentInstrument` / `fetchPaymentInstrument` / `deletePaymentInstrument` manage the reusable instrument (`instr_` prefix) independently of the agreement. Before deleting, confirm no active agreement or pending charge depends on it — the API will not do that check for you.
