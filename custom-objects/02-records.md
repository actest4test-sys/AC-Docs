---
title: "Step 2 — Records"
layout: default
parent: Custom Objects
nav_order: 2
permalink: /custom-objects/02-records/
---

# Step 2 — Records

Records are the instances of a schema. Each record sets values for the schema's fields and (typically) links to a contact via the `primary-contact` relationship.

![Custom object records shown in the ActiveCampaign UI]({{ site.baseurl }}/assets/images/custom-object-records.jpg)
*Custom object records shown in the ActiveCampaign UI.*

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
- **`fields`** — array of `{id, value}` objects. The `id` must match a `field.id` from the schema (Step 1).
- **`relationships`** — an object keyed by relationship ID (`primary-contact` is the common case). The value is an array of related contact IDs as strings (e.g. `["242"]`).

### Updating a record

Custom object records are updated by re-POSTing to the same schema endpoint with the same `externalId`. The API performs an upsert: if a record with that `externalId` already exists, it is updated; otherwise a new record is created.

> **Send all fields on every update.** The upsert replaces the record's field values with exactly what you send. Any field you omit will be **cleared** — its value will be removed from the record. Always include the full set of field values in every POST, not just the fields that changed.
>
> Verified against the live API: a record created with four fields (`amount`, `donation-date`, `campaign-name`, `payment-method`) was re-POSTed with only two of those fields; a subsequent GET showed only those two fields present — the other two were cleared.

The `PUT /api/3/customObjects/records/{schemaId}/{recordId}` path returns `405 Method Not Allowed` — the correct update path is `POST /api/3/customObjects/records/{schemaId}` with the `externalId` in the body.

### Response

`HTTP 201 Created`. The body echoes the record back:

```json
{
  "record": {
    "externalId": "donation-12345",
    "schemaId": "4453571f-4a21-45e9-9872-49ce0f86e611",
    "fields": [
      {"id": "amount", "value": "50"},
      {"id": "donation-date", "value": "2026-05-21T00:00:00Z"},
      {"id": "campaign-name", "value": "Spring Appeal"},
      {"id": "payment-method", "value": "Credit Card"}
    ],
    "relationships": { "primary-contact": ["242"] }
  }
}
```

> **Note:** the `201` response does not include the AC-assigned record `id`. To retrieve it, list the records ([Step 3](../03-querying)) and match on your `externalId`.

## Linking to a contact

The contact must exist in AC before you can reference it. If you're integrating from an external system, sync the contact first via [`POST /api/3/contact/sync`](../../contacts-lists-tags/01-sync-contact/) and use the returned `contact.id` in the `relationships.primary-contact` array.

> **⚠ Eventual consistency**
>
> A record returned with `201 Created` may not appear in the records-list endpoint (Step 3) for several seconds. If you create a record and immediately list to verify, you may see a stale view and incorrectly conclude the create failed. **Trust the `201` response.** If you must verify, wait at least 10 seconds before re-querying.

---

Next: [Step 3 — Querying](../03-querying)
