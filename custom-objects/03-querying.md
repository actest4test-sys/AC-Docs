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
GET https://{youraccountname}.api-us1.com/api/3/customObjects/records/{schemaId}
```

Returns paginated records for the schema. Default ordering is **not guaranteed** — use `orders[createdTimestamp]` (below) for explicit sorting.

### Response shape

```json
{
  "records": [
    {
      "id": "27783bc8-beff-42dc-a1c8-cbb8e1c84ccb",
      "externalId": "np-donation-3f7269d8",
      "schemaId": "4453571f-4a21-45e9-9872-49ce0f86e611",
      "fields": [
        {"id": "donation-date", "value": "2026-04-15T00:00:00Z"},
        {"id": "amount", "value": "250"},
        {"id": "campaign-name", "value": "Spring Appeal 2026"},
        {"id": "is-recurring", "value": "Yes"},
        {"id": "payment-method", "value": "Direct Debit"}
      ],
      "relationships": {
        "primary-contact": ["892"]
      },
      "createdTimestamp": "2026-05-05T13:06:10.520Z",
      "updatedTimestamp": "2026-05-05T13:06:10.520Z"
    }
  ],
  "meta": {"total": 12, "count": 1, "limit": 100, "offset": 0}
}
```

## Filter by exact createdTimestamp

```
GET https://{youraccountname}.api-us1.com/api/3/customObjects/records/{schemaId}?filters[createdTimestamp]=2026-05-21T11:14:25.386Z
```

Exact match only — pass the full ISO 8601 timestamp including milliseconds. Returns the matching record(s).

> **⚠ No range / before / after filter for `createdTimestamp`**
>
> Variants like `filters[created_before]`, `filters[created_after]`, `filters[createdTimestamp_gt]`, and `filters[createdTimestamp][gt]` all return `total: 0` rather than a date-range result. To do a date-range query, sort with `orders[createdTimestamp]`, paginate, and break the loop in client code when records cross your cutoff.

## Get a single record by ID

```
GET https://{youraccountname}.api-us1.com/api/3/customObjects/records/{schemaId}/{recordId}
```

Returns the full record (including `id`, `createdTimestamp`, `updatedTimestamp`) wrapped under `record`:

```json
{
  "record": {
    "id": "27783bc8-beff-42dc-a1c8-cbb8e1c84ccb",
    "externalId": "np-donation-3f7269d8",
    "schemaId": "4453571f-4a21-45e9-9872-49ce0f86e611",
    "fields": [
      {"id": "donation-date", "value": "2026-04-15T00:00:00Z"},
      {"id": "amount", "value": "250"},
      {"id": "campaign-name", "value": "Spring Appeal 2026"},
      {"id": "is-recurring", "value": "Yes"},
      {"id": "payment-method", "value": "Direct Debit"}
    ],
    "relationships": {"primary-contact": ["892"]},
    "createdTimestamp": "2026-05-05T13:06:10.520Z",
    "updatedTimestamp": "2026-05-05T13:06:10.520Z"
  }
}
```

## Deleting a record

```
DELETE https://{youraccountname}.api-us1.com/api/3/customObjects/records/{schemaId}/{recordId}
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
