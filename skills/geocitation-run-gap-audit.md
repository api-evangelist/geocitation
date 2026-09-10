---
name: Run a GEOCitation Gap-Analysis audit for a URL
description: Launch a Gap-Analysis AEO/GEO audit for your own URL against a keyword, track it to completion, and retrieve the citation-gap report and semantic blueprint.
api: openapi/geocitation-openapi.json
operations: [create_audit_v1_audits_post, list_audit_events_v1_audits__audit_id__events_get, get_audit_v1_audits__audit_id__get, get_document_v1_audits__audit_id__documents__doc_type__get, read_usage_v1_usage_get]
---

# Run a GEOCitation Gap-Analysis audit

Base URL: `https://api.geocitation.io/v1`. Auth header: `X-API-Key: geo_citation_...`.

## Steps

1. **Check quota** — `GET /v1/usage` (`read_usage_v1_usage_get`) returns
   `{plan, used, limit, remaining}`. If `remaining` is 0 the launch will `402`.
2. **Launch the Gap audit** — `POST /v1/audits` (`create_audit_v1_audits_post`)
   with `{"audit_type": "gap", "keyword": "<keyword>", "user_url":
   "https://your-site.example/page", "language": "en", "country": "US"}`.
   Returns **202** + `audit_id`. `user_url` must be a valid URI (≤2083 chars) and
   must not be on the opt-out list.
3. **Hydrate / follow events** — `GET /v1/audits/{audit_id}/events`
   (`list_audit_events_v1_audits__audit_id__events_get`) replays the node event
   log for a frontend; the SSE stream gives live updates. A failed document node
   can be re-run with `POST /v1/audits/{audit_id}/retry-doc/{doc_slot}`.
4. **Retrieve output** — when `status` is `completed`, `GET /v1/audits/{audit_id}`
   (`get_audit_v1_audits__audit_id__get`) for the JSON, then
   `GET /v1/audits/{audit_id}/documents/{doc_type}`
   (`get_document_v1_audits__audit_id__documents__doc_type__get`) for Gap docs:
   `citation_gap_report`, `semantic_blueprint`, `laser_execution_outline`. Use
   `GET /v1/audits/{audit_id}/manifest` for the SOC2 bank-proof manifest.

## Rules

- Poll `status` or subscribe to the `audit.completed` / `audit.failed` webhook
  (verify `X-GEOCitation-Signature`, HMAC-SHA256 `sha256=<hexdigest>` over the raw
  body) rather than fetching documents before completion — a premature fetch `409`s.
- No idempotency key: one successful `POST /v1/audits` spends one audit.
