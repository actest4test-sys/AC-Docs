---
title: "Bulk import"
layout: default
parent: "Step 1 — Sync a contact"
grand_parent: "Contacts, Lists & Tags"
nav_order: 1
permalink: /contacts-lists-tags/01-sync-contact/bulk-import/
---

# Bulk import

For batch backfills, historical-data loads, or any sync where you have **more than a handful of contacts**, use the bulk import endpoint instead of looping `POST /contact/sync`. It's asynchronous, accepts up to 250 contacts per request, and avoids per-contact rate limits.

```
POST /api/3/import/bulk_import
```

---

## When to use it (vs `/contact/sync`)

| | `/contact/sync` | `/import/bulk_import` |
|---|---|---|
| Mode | Synchronous | **Asynchronous** (queued) |
| Per-call payload | 1 contact | Up to **250 contacts** |
| Response time | Returns the contact | Returns immediately with a `batchId` |
| Rate limit | 5 req/sec (standard API) | 100 req/min multi-contact; 20 req/min single-contact (separate import quota) |
| Field naming | **camelCase** (`firstName`) | **snake_case** (`first_name`) |
| Use for | Real-time integration | Migrations, nightly syncs, CSV imports |

---

## Request body

```json
{
  "contacts": [
    {
      "email": "jane@example.com",
      "first_name": "Jane",
      "last_name": "Doe",
      "phone": "+353 87 123 4567",
      "tags": ["EU Lead", "Spring Campaign 2026"],
      "subscribe": [{"listid": 3}],
      "fields": [
        {"id": 1, "value": "Premium"},
        {"id": 5, "value": "Kildare"}
      ]
    }
  ],
  "callback": {
    "url": "https://example.com/ac-callback",
    "requestType": "POST",
    "params": [
      {"key": "secret", "value": "shared-token-here"}
    ]
  },
  "exclude_automations": false
}
```

---

## Response

### Success — `HTTP 200`

```json
{
  "success": 1,
  "queued_contacts": 1,
  "batchId": "37fbcf9a-02a7-4f8e-a9ea-6f50b1085e81",
  "message": "Contact import queued"
}
```

The response confirms the batch was **queued**, not processed. The actual contact rows appear a few seconds later (in this session's probe: ~6 seconds for a 1-contact batch).

### Validation error — `HTTP 400`

When the payload is malformed or contacts are missing required fields:

```json
{
  "success": 0,
  "message": "JSON payload did not pass validation. Please fix failureReasons and retry. The import was not queued for processing.",
  "failureReasons": [
    {"contact": 1, "failureReason": "Field 'email' not found"}
  ]
}
```

> **⚠ All-or-nothing validation:** if **any** contact in the batch fails validation, the **entire batch is rejected** — none are queued. Either scrub the payload before sending, or accept the rejection and retry with the valid subset.

`failureReasons` takes one of two forms depending on the error type:
- A **string array** for top-level/general errors (e.g. `["Rate limit exceeded."]`)
- An **object array** for per-contact validation errors (`[{"contact": <index>, "failureReason": "..."}]`, 1-indexed)

---

## Limits

- **250 contacts** per request — chunk larger imports into pages of 250.
- **400 KB** maximum payload size per request (399,999 bytes or less).
- **Rate limit: 100 requests per minute** for multi-contact batches; **20 requests per minute** for single-contact batches (separate quota from the main REST API). At 100 req/min with 250 contacts per request, theoretical max ≈ 25,000 contacts/minute. Build in delay if you hit rate limits.
- Asynchronous processing — the `batchId` is your only handle on the batch. Use the status endpoint or the `callback` URL to verify completion.

---

## Checking batch status

Use the bulk import status endpoint to check whether a queued batch has finished and see which contacts succeeded or failed.

```
GET /api/3/import/info?batchId=<batchId>
```

**Example response (completed):**

```json
{
  "status": "completed",
  "success": ["1250", "1251", "1252"],
  "failure": []
}
```

- `success` — array of AC contact IDs (as strings) for contacts that were created or updated.
- `failure` — array of email addresses that failed to import (e.g. invalid format, on exclusion list, bounced).
- `status` can be: `waiting`, `claimed`, `active`, `completed`, `failed`, or `interrupted`.

> **Note:** If you call this endpoint less than a second after submitting the import, `status` may return `false` (a boolean) because the system has not yet assigned a status to the batch. Wait at least a few seconds before polling.

**Example response (bad/missing batchId — `HTTP 400`):**

```json
{
  "success": 0,
  "message": "'batchId' is a required field."
}
```

See the [official reference](https://developers.activecampaign.com/reference/bulk-import-status-info) for full detail.

---

## Pitfalls

- **camelCase vs snake_case.** `/contact/sync` uses `firstName`; `/import/bulk_import` uses `first_name`. Mixing them silently drops the values (no validation error — the field is just unknown).
- **One bad contact rejects the entire batch.** Pre-validate emails client-side before sending. (See the callout in [Validation error](#validation-error--http-400) above.)
- **Tag names auto-create tags.** A typo in `"Premium"` vs `"premuim"` creates a new permanent tag in the account. Hard to clean up later — normalize tag names before sending.
- **`fields[].id` is numeric.** Same gotcha as `/contact/sync` and `POST /fieldValues` — `perstag` doesn't work here.
- **No idempotency by external ID.** Unlike ecommerce records, bulk_import has no `externalid` field. Dedup is by email only — re-importing the same email updates the existing contact.
- **Automations fire by default.** If you're loading historical data, set `exclude_automations: true` so contacts don't get re-enrolled in welcome/onboarding flows.
- **Don't immediately re-query after `success: 1`.** The batch is queued, not done. Allow at least 5–10 seconds before checking status. Premature verification leads to thinking the import failed and double-importing.

---

## Reference

- [Bulk Import (AC developer docs)](https://developers.activecampaign.com/reference/bulk-import-contacts)
- [Bulk Import Status Info (AC developer docs)](https://developers.activecampaign.com/reference/bulk-import-status-info)

---

Back to: [Step 1 — Sync a contact](../)
