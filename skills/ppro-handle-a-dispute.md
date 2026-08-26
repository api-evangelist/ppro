---
name: ppro-handle-a-dispute
description: Work a PPRO dispute end to end — read the case, respect its capability gates, submit evidence, action it, and reconcile the resulting chargeback.
api: PPRO Global API
base_url: https://api.eu.ppro.com
operations:
  - getDisputes
  - getById
  - postMessage
  - uploadFile
  - downloadFile
  - actionDispute
  - getChargebacks
  - getChargeback
  - getChargebackReversals
  - disputeReports
  - getReportById
generated: '2026-08-26'
method: generated
source: openapi/ppro-risk-management-openapi.yml, openapi/ppro-chargebacks-openapi.yml, openapi/ppro-dispute-reports-openapi.yml, https://developerhub.ppro.com/global-api/docs/disputes
---

# Handle a PPRO dispute

Verified against `openapi/ppro-risk-management-openapi.yml`, `openapi/ppro-chargebacks-openapi.yml` and `openapi/ppro-dispute-reports-openapi.yml`.

## Step 1 — find the dispute

`getDisputes` — `GET /v1/disputes` — returns every dispute against a PPRO payment charge ID.
`getById` — `GET /v1/disputes/{disputeId}` — returns one case.

## Step 2 — read the capabilities before doing anything

The dispute carries a capability list, and it is a hard gate, not advice. `postMessage` works only when `POST MESSAGE` is allowed; `uploadFile` only when `UPLOAD_FILE` is allowed. Check first — the scheme's phase, not your intent, decides what is possible.

## Step 3 — attach evidence

`uploadFile` — `POST /v1/disputes/{disputeId}/files`

`multipart/form-data`, with the `file` part carrying **raw binary** — PDF or image bytes. Base64 is explicitly unsupported, including via `Content-Transfer-Encoding: base64`; omit that header or set it to `binary`. Set a real `Content-Type` on the part (`application/pdf`, `image/png`).

`downloadFile` — `GET /v1/disputes/{disputeId}/files/{fileId}` — returns the file as a binary stream.

`postMessage` — `POST /v1/disputes/{disputeId}/messages` — adds narrative to the case.

## Step 4 — action it

`actionDispute` — `POST /v1/disputes/{disputeId}/actions` — accept, challenge or offer.

This is a terminal merchant decision routed to the scheme. There is no reversal endpoint. Get the evidence in first.

PPRO's dispute lifecycle covers pre-disputes, pre-arbitration and (since May 2026) arbitration, so a case may move through more phases than a single accept/challenge implies.

## Step 5 — reconcile the money

`getChargebacks` — `GET /v1/chargebacks` — requires exactly one of `disputeId` or `paymentChargeId`, never both.
`getChargeback` — `GET /v1/chargebacks/{id}`
`getChargebackReversals` / `getChargebackReversal` — the reversal in your favour, same one-of-two-filters rule.

Listen for `CHARGEBACK_SUCCEEDED`.

## Step 6 — report

`disputeReports` — `POST /v1/dispute-reports` — asynchronous. The date range must sit inside the last 90 days and end before now; anything wider is rejected.

`getReportById` — `GET /v1/dispute-reports/{reportId}` — returns metadata, and a pre-signed download URL once processing completes. Watch for `REPORT_PROCESSED`, `REPORT_FAILED` and `REPORT_EXPIRED`; the download URL does not live forever.
