---
name: Run a GEOCitation Market audit and retrieve the report
description: Launch a Market-Intelligence AEO/GEO audit for a keyword, wait for completion, and pull the structured JSON output and Markdown deliverables.
api: openapi/geocitation-openapi.json
operations: [create_audit_v1_audits_post, get_audit_status_v1_audits__audit_id__status_get, get_audit_v1_audits__audit_id__get, get_document_v1_audits__audit_id__documents__doc_type__get]
---

# Run a GEOCitation Market audit

Base URL: `https://api.geocitation.io/v1`. Authenticate every request with
`X-API-Key: geo_citation_...` (a Bearer token is also accepted).

## Steps

1. **Launch the audit** — `POST /v1/audits` (`create_audit_v1_audits_post`) with
   `{"audit_type": "market", "keyword": "<2-200 chars>", "language": "en",
   "country": "US", "intent": "auto"}`. Supported countries: BE, CA, CH, DE, ES,
   FR, GB, JP, MA, SN, US. The call returns **202** with an `audit_id`. A `422`
   means an invalid `audit_type`, keyword length, or country code — fix the input.
2. **Wait for completion** — poll `GET /v1/audits/{audit_id}/status`
   (`get_audit_status_v1_audits__audit_id__status_get`) and watch `status` /
   `progress_pct`. Completion typically takes 5-7 minutes. Prefer the SSE stream
   `GET /v1/audits/{audit_id}/stream`, or a signed `audit.completed` webhook, over
   tight polling. Respect rate limits: 100 requests/minute per key.
3. **Fetch the full output** — once `status` is `completed`, call
   `GET /v1/audits/{audit_id}` (`get_audit_v1_audits__audit_id__get`) for the full
   structured JSON (`output` is `null` until completed).
4. **Pull deliverables** — `GET /v1/audits/{audit_id}/documents/{doc_type}`
   (`get_document_v1_audits__audit_id__documents__doc_type__get`) for Market docs:
   `content_gap_report`, `remediation_blueprint`, `laser_optimization_brief`. A
   `409` means the audit is not yet `completed`; a `404` means the id is not yours.

## Rules

- Errors use `{"detail": "..."}` (not RFC 9457). A `402` carries
  `{"detail": {"error": "quota_exceeded", ...}}` — check `GET /v1/usage` first.
- No idempotency key exists: do not blindly retry `POST /v1/audits`, or you spend
  another audit from your quota. On `429`, honor the `Retry-After` header.
