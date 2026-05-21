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

Returns paginated records for the schema, ordered by creation time. **The schema ID goes in the URL path** — other URL shapes (`/customObjects/records?schemaId=...`, `/customObjects/{schemaId}/records`) return `400 Bad Request`.

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

## Get a single record by ID

The most reliable approach is to filter the list response on `id` or `externalId`. A direct `GET /customObjects/records/{schemaId}/{recordId}` exists per the API reference; behaviour and shape can vary by tenant — check the [Custom Objects API reference](https://developers.activecampaign.com/reference/customobjects) for your account.

## Updating and deleting records

- **Update:** `PATCH /api/3/customObjects/records/{schemaId}/{recordId}` with a partial body shaped like Step 2.
- **Delete:** `DELETE /api/3/customObjects/records/{schemaId}/{recordId}`.

See the API reference linked above for exact body shapes and edge cases.

> **⚠ Don't trust the list endpoint to confirm a just-created record**
>
> Due to eventual consistency, a record you just `POST`ed may not be in the next list response. Trust the `201` from the POST. If you re-query and "miss" the record, you risk re-creating it — a duplicate.

---

Back to: [Custom Objects overview](../)
