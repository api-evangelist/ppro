# PPRO

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

PPRO is a local payments infrastructure company. It lets payment service providers, acquirers,
platforms and enterprise merchants accept the payment methods consumers actually use in their own
market — bank redirects (iDEAL, BLIK, Przelewy24, Pay By Bank), wallets (Alipay, WeChat Pay, Amazon
Pay, Cash App Pay, TWINT, Swish), cash and voucher rails (Boleto, OXXO, Multibanco, Indomaret), SEPA
Direct Debit, Pix, UPI, BNPL and stablecoins — through one REST integration.

## What this profile found

- **Eleven OpenAPI 3.1 documents, 63 operations and 30 webhook definitions**, all served from PPRO's
  own developer hub and discovered through an RFC 9727 API catalog at
  `https://developerhub.ppro.com/.well-known/api-catalog`.
- **A live, first-party remote MCP server** at `https://mcp.eu.ppro.com` (sandbox
  `https://mcp.sandbox.eu.ppro.com`). Anonymous `initialize` + `tools/list` returned **28 tools with
  complete JSON Schema inputs**, saved verbatim in `mcp/ppro-mcp-tools.json`. Every tool binds
  one-to-one to a published OpenAPI `operationId`.
- **A documented idempotency contract** (`Request-Idempotency-Key`, UUIDv4, 24h retention, 409 on
  in-flight or body-mismatch replay) and a **token-bucket rate limit with four response headers**.
- **CloudEvents 1.0.2 webhooks** with HMAC-SHA256 `PPRO-Signature` verification and a documented
  retry ladder (15s first retry, doubling, 15 attempts, ~68 hours).
- **76 published failure codes** across four failure types, catalogued in
  `errors/ppro-failure-codes.yml`.
- **No AsyncAPI, no GraphQL, no gRPC, no SOAP, no OAuth, no agent card, no security.txt** — all
  probed, all recorded as absent rather than assumed.

## Contents

| Directory | What is in it |
|---|---|
| `openapi/` | Eleven PPRO OpenAPI 3.1 documents; raw harvests in `_original/` |
| `overlays/` | OpenAPI Overlay 1.0.0 documents carrying our enhancements |
| `mcp/` | MCP server manifest, the verbatim `tools/list` response, and the tool ↔ REST crosswalk |
| `asyncapi/` | The 30-event CloudEvents webhook catalogue |
| `conventions/` | Auth, idempotency, pagination, data standards, reversibility |
| `errors/` | HTTP problem types and the 76 published failure codes |
| `authentication/`, `security/` | Auth profile, domain security probe, trust center, disclosure probe |
| `lifecycle/`, `changelog/` | Versioning and deprecation posture, monthly changelog |
| `sandbox/`, `plans/`, `rate-limits/` | Test environment and triggers, pricing posture, published limits |
| `packages/`, `components/` | The one first-party npm SDK, and the Drop-in Checkout component family |
| `data-model/` | 19-entity graph derived from the `$ref` and id-reference links |
| `conformance/` | Standards conformance, including the domain-standard signatures |
| `skills/`, `llms/`, `well-known/` | Agent skills, PPRO's own llms.txt files, well-known probe record |

## Primary public sources

- https://www.ppro.com/
- https://developerhub.ppro.com/
- https://developerhub.ppro.com/global-api/reference
- https://developerhub.ppro.com/.well-known/api-catalog
- https://developerhub.ppro.com/llms.txt
- https://mcp.eu.ppro.com
- https://status.ppro.com/
- https://trust.ppro.com/
- https://www.postman.com/pprodev (linked from PPRO's own developer-resources page)
- https://www.npmjs.com/package/@pprogroup/drop-in-checkout
