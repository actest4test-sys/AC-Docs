---
title: "Step 3 — Querying"
layout: default
parent: Custom Objects
nav_order: 3
permalink: /custom-objects/03-querying/
---

# Step 3 — Querying

## List all records for a schema

```
GET /api/3/customObjects/records/{schemaId}?limit=100
```

Returns paginated records for the schema. **The schema ID goes in the URL path** — other URL shapes (`/customObjects/records?schemaId=...`, `/customObjects/{schemaId}/records`) return `400 Bad Request`. Default ordering is **not guaranteed** — use `orders[createdTimestamp]` (below) for explicit sorting.

### Response shape

```json
{
  "records": [
    {
      "id": "c7d03114-338e-439b-9380-91122fa25ddd",
      "externalId": "donation-12345",
      "schemaId": "4453571f-4a21-45e9-9872-49ce0f86e611",
      "fields": [
        {"id": "amount", "value": "50"},
        {"id": "donation-date", "value": "2026-05-21T00:00:00Z"}
      ],
      "relationships": {
        "primary-contact": ["242"]
      },
      "createdTimestamp": "2026-05-21T11:14:25.386Z",
      "updatedTimestamp": "2026-05-21T11:14:25.386Z"
    }
  ],
  "meta": {"total": 10, "count": 10, "limit": 100, "offset": 0}
}
```

## Filter records for a specific contact

The records-list endpoint does not expose a `relatedId` filter. The pragmatic pattern is to fetch the page and filter in code:

```python
records = response["records"]
for_contact = [
    r for r in records
    if str(contact_id) in r.get("relationships", {}).get("primary-contact", [])
]
```

For schemas with many records, paginate with `limit` + `offset`:

```
GET /api/3/customObjects/records/{schemaId}?limit=100&offset=100
```

`meta.total` in the response tells you when you've reached the end.

## Sort by createdTimestamp

```
GET /api/3/customObjects/records/{schemaId}?orders[createdTimestamp]=ASC
GET /api/3/customObjects/records/{schemaId}?orders[createdTimestamp]=DESC
```

Server-side sort by record creation time. The most useful pattern for "give me everything in chronological order" or "give me the newest N records first".

## Filter by exact createdTimestamp

```
GET /api/3/customObjects/records/{schemaId}?filters[createdTimestamp]=2026-05-21T11:14:25.386Z
```

Exact match only — pass the full ISO 8601 timestamp including milliseconds. Returns the matching record(s).

> **⚠ No range / before / after filter for `createdTimestamp`**
>
> Variants like `filters[created_before]`, `filters[created_after]`, `filters[createdTimestamp_gt]`, and `filters[createdTimestamp][gt]` all return `total: 0` rather than a date-range result. To do a date-range query, sort with `orders[createdTimestamp]`, paginate, and break the loop in client code when records cross your cutoff.

## Get a single record by ID

```
GET /api/3/customObjects/records/{schemaId}/{recordId}
```

Returns the full record (including `id`, `createdTimestamp`, `updatedTimestamp`) wrapped under `record`:

```json
{
  "record": {
    "id": "c7d03114-338e-439b-9380-91122fa25ddd",
    "externalId": "donation-12345",
    "schemaId": "4453571f-4a21-45e9-9872-49ce0f86e611",
    "fields": [ ... ],
    "relationships": {"primary-contact": ["242"]},
    "createdTimestamp": "2026-05-21T11:14:25.386Z",
    "updatedTimestamp": "2026-05-21T11:14:25.386Z"
  }
}
```

## Deleting a record

```
DELETE /api/3/customObjects/records/{schemaId}/{recordId}
```

Returns `HTTP 202 Accepted` with an empty body. The record is removed immediately from subsequent list and single-GET responses.

> **⚠ Updating records is not supported via PATCH / PUT**
>
> Both `PATCH /api/3/customObjects/records/{schemaId}/{recordId}` and `PUT /api/3/customObjects/records/{schemaId}/{recordId}` return `405 Method Not Allowed`. To "edit" a record, delete and recreate, or update from the AC UI. Check the [Custom Objects API reference](https://developers.activecampaign.com/reference/custom-objects) periodically — the update endpoint may be added later.

> **⚠ Don't trust the list endpoint to confirm a just-created record**
>
> Due to eventual consistency, a record you just `POST`ed may not be in the next list response. Trust the `201` from the POST. If you re-query and "miss" the record, you risk re-creating it — a duplicate.

---

Back to: [Custom Objects overview](../)
