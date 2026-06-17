---
title: "Step 1 — Schemas"
layout: default
parent: Custom Objects
nav_order: 1
permalink: /custom-objects/01-schemas/
---

# Step 1 — Schemas

A schema defines the shape of a custom object — its name, its fields, and what it relates to. Schemas are usually created in the AC UI; the API is most useful for **discovering** existing schemas and **inspecting** their fields before writing records against them.

## List all schemas

```
GET /api/3/customObjects/schemas
```

Returns every schema in the account: slug, ID, singular/plural labels, fields, relationships.

### Example response (trimmed)

Check the full example response [here](https://developers.activecampaign.com/reference/list-all-schemas).

```json
{
  "schemas": [
    {
      "id": "4453571f-4a21-45e9-9872-49ce0f86e611",
      "slug": "np-donation",
      "visibility": "private",
      "customized": false,
      "labels": {"singular": "Donation", "plural": "Donations"},
      "description": "Donor records linked to contact",
      "createdTimestamp": "2026-05-05T13:00:24.898989309Z",
      "updatedTimestamp": "2026-05-05T13:00:24.898989309Z",
      "fields": [
        {
          "id": "amount",
          "labels": {"singular": "Amount", "plural": "Amounts"},
          "type": "text",
          "required": false,
          "inherited": false
        },
        {
          "id": "donation-date",
          "labels": {"singular": "Donation Date", "plural": "Donation Dates"},
          "type": "datetime",
          "required": false,
          "inherited": false
        }
      ],
      "icons": {
        "default": "https://d226aj4ao1t61q.cloudfront.net/n9mayqo2d_customobject.png"
      },
      "relationships": [
        {
          "id": "primary-contact",
          "labels": {"singular": "Contact", "plural": "Contacts"},
          "description": "Primary contact linked to this record",
          "namespace": "contacts",
          "hasMany": false,
          "inherited": false
        }
      ]
    }
  ],
  "meta": {
    "total": 58,
    "count": 20,
    "limit": 20,
    "offset": 0
  }
}
```

## Get a single schema

You can find the call to get a single schema (GET `/api/3/customObjects/schemas/{id}`) [here](https://developers.activecampaign.com/reference/retrieve-a-schema).

## Creating a schema via API

Schemas are normally set up in the AC UI, but the API does support creation:

```
POST /api/3/customObjects/schemas
```

The request body below is trimmed — see the full reference [here](https://developers.activecampaign.com/reference/retrieve-a-schema).

```json
{
  "schema": {
    "slug": "establishment",
    "labels": {"singular": "Establishment", "plural": "Establishments"},
    "description": "Hotel, B&B, restaurant or other inspected place",
    "fields": [
      {"id": "establishment-name", "labels": {"singular": "Name", "plural": "Names"}, "type": "text", "required": false},
      {"id": "last-inspection-date", "labels": {"singular": "Last Inspection", "plural": "Last Inspections"}, "type": "datetime", "required": false}
    ],
    "relationships": [
      {
        "id": "primary-contact",
        "labels": {"singular": "Contact", "plural": "Contacts"},
        "namespace": "contacts",
        "hasMany": false
      }
    ]
  }
}
```

---

Next: [Step 2 — Records](../02-records)
