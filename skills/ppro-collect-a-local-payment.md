---
name: ppro-collect-a-local-payment
description: Collect a one-off payment from a consumer with a PPRO local payment method or card, drive the consumer through authentication, and capture the funds.
api: PPRO Global API
base_url: https://api.eu.ppro.com
sandbox_url: https://api.sandbox.eu.ppro.com
operations:
  - authorize
  - confirm
  - getPaymentCharge
  - capture
  - getCapture
  - getCaptures
generated: '2026-08-26'
method: generated
source: openapi/ppro-payment-charges-openapi.yml, conventions/ppro-conventions.yml, https://developerhub.ppro.com/global-api/docs/quickstart-lpms
---

# Collect a local payment with PPRO

Every operationId below is verified against `openapi/ppro-payment-charges-openapi.yml`. Do not invent endpoints.

## Before you start

- Authenticate with `Authorization: Bearer {API_KEY}`. Sandbox keys work only against `https://api.sandbox.eu.ppro.com`.
- Send `Merchant-Id: {MERCHANT_ID}` — payment endpoints require it. Do not send it where the operation does not ask for it; PPRO warns that unnecessary headers can fail the request.
- Generate a fresh UUIDv4 and send it as `Request-Idempotency-Key` on every POST. Keep it for 24 hours so a retry after a timeout replays instead of double-charging.
- Amounts are integers in ISO 4217 minor units. `10.00 EUR` is `{"value": 1000, "currency": "EUR"}`.

## Step 1 — create the charge

`authorize` — `POST /v1/payment-charges`

Send `amount`, `country`, `paymentMethod`, `paymentMedium`, the `consumer` block, a `merchantPaymentChargeReference` you can reconcile against, `authenticationSettings` carrying your `returnUrl`, and `webhooksUrl` if you are not using an account-level default.

Set `autoCapture: true` when you want funds captured immediately and have no reason to hold the authorization. Leave it off when you ship later — that is what makes Step 4 a decision rather than a formality.

The response is a charge with an `id` prefixed `charge_` and a `status`.

## Step 2 — send the consumer through authentication

If `status` is `AUTHENTICATION_PENDING`, the charge carries the authentication instructions for the flow the payment method uses: `REDIRECT`, `SCAN_CODE`, `MULTI_FACTOR`, `APP_INTENT`, `APP_NOTIFICATION` or `EXTERNAL_3DS`. Follow the flow, not the payment method — PPRO's docs are explicit that method-specific branching is what stops an integration scaling.

If the flow needs more data from you before it can proceed, call `confirm` — `POST /v1/payment-charges/{paymentChargeId}/authorizations`.

## Step 3 — learn the outcome from the webhook, not a poll

Handle `PAYMENT_CHARGE_AUTHORIZATION_SUCCEEDED` and `PAYMENT_CHARGE_AUTHORIZATION_FAILED`. Verify the `PPRO-Signature` header before trusting the body: compute `sha256(t + "." + rawPayload)` keyed with your signing secret, where `t` is the timestamp in the header, and compare against `s`. Acknowledge with a 2xx immediately and store before processing.

`getPaymentCharge` — `GET /v1/payment-charges/{paymentChargeId}` — is the reconciliation call, not the primary signal.

On failure, read the `failure` object. `failureType` tells you whose decision it was (`INTERNAL_DECLINE`, `PROVIDER_DECLINE`, `INTERNAL_ERROR`, `PROVIDER_ERROR`) and `failureCode` names the reason — the 76 published codes and their recommended handling are in `errors/ppro-failure-codes.yml`. Only retry when `isRetryable` is true.

## Step 4 — capture

`capture` — `POST /v1/payment-charges/{paymentChargeId}/captures` with the `amount` and an optional `merchantCaptureReference`.

Capture in full, partially, or repeatedly up to the authorized amount. Confirm with `getCapture` / `getCaptures` before you treat the money as collected — a capture can come back `FAILED` inside a 200 response.

## What can be taken back

- Before capture: `processVoid` — `POST /v1/payment-charges/{paymentChargeId}/voids`.
- After capture: `refund` — `POST /v1/payment-charges/{paymentChargeId}/refunds`.
- Once the charge is `CAPTURED` it can no longer be voided.
- PPRO does not publish a platform-wide refund window; validity periods are stated per payment method. Check the payment method's page before promising a customer a refund is possible.

## Errors

`409` on a POST means an idempotency-key conflict — the same key is in flight, or was reused with a different body. Do not treat it as a failed payment. `429` means the token bucket is empty; back off using `X-RateLimit-Replenish-Rate`. `504` is the case idempotency exists for: retry with the same key.
