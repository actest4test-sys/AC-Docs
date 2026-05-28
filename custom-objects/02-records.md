---
title: "Step 2 — Records"
layout: default
parent: Custom Objects
nav_order: 2
permalink: /custom-objects/02-records/
---

# Step 2 — Records

Records are the instances of a schema. Each record sets values for the schema's fields and (typically) links to a contact via the `primary-contact` relationship.

## Create a record

```
POST /api/3/customObjects/records/{schemaId}
```

> **Note the path** — the endpoint is `/records/{schemaId}`, not `/records?schemaId=...`. The schema UUID goes in the URL path, not the query string. The plain `/records` endpoint returns `400 Bad Request`.

### Request body

```json
{
  "record": {
    "externalId": "donation-12345",
    "fields": [
      {"id": "amount", "value": "50"},
      {"id": "donation-date", "value": "2026-05-21T00:00:00Z"},
      {"id": "campaign-name", "value": "Spring Appeal"},
      {"id": "payment-method", "value": "Credit Card"}
    ],
    "relationships": {
      "primary-contact": ["242"]
    }
  }
}
```

### Key fields

- **`externalId`** — optional but recommended. Your system's unique ID for the record. Use it for idempotent re-syncs.
- **`fields`** — array of `{id, value}` objects. The `id` must match a `field.id` from the schema (Step 1). Unknown field IDs are silently dropped.
- **`relationships`** — an object keyed by relationship ID (`primary-contact` is the common case). The value is **an array of related IDs as strings** — even when the relationship is `hasMany: false`, the value is a single-element array.

### Field value types

| Schema `field.type` | Body value |
|---|---|
| `text` | string (numeric values like ratings/amounts also go in as strings, e.g. `"50"`) |
| `datetime` | ISO 8601 with `Z` or offset (e.g. `2026-05-21T00:00:00Z`) |

### Response

`HTTP 201 Created`. The body echoes the record back — but **note the `id` field is not included**:

```json
{
  "record": {
    "externalId": "donation-12345",
    "schemaId": "4453571f-4a21-45e9-9872-49ce0f86e611",
    "fields": [ ... ],
    "relationships": { "primary-contact": ["242"] }
  }
}
```

> **⚠ POST does not return the new record's `id`**
>
> To get the AC-assigned UUID for a record you just created, either list the records ([Step 3](../03-querying)) and match on your `externalId`, or `GET` the single record by ID once you know it from a subsequent list. Track your own `externalId` carefully — it's the only handle you have on the record immediately after creation.

## Linking to a contact

The contact must exist in AC before you can reference it. If you're integrating from an external system, sync the contact first via [`POST /api/3/contact/sync`](../../contacts-lists-tags/01-sync-contact/) and use the returned `contact.id` in the `relationships.primary-contact` array.

> **⚠ Eventual consistency**
>
> A record returned with `201 Created` may not appear in the records-list endpoint (Step 3) for several seconds. If you create a record and immediately list to verify, you may see a stale view and incorrectly conclude the create failed. **Trust the `201` response.** If you must verify, wait at least 10 seconds before re-querying.

---

Next: [Step 3 — Querying](../03-querying)
