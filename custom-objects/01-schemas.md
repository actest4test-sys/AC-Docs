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
GET /api/3/customObjects/schemas?limit=100
```

Returns every schema in the account: slug, ID, singular/plural labels, fields, relationships.

### Example response (trimmed)

```json
{
  "schemas": [
    {
      "id": "4453571f-4a21-45e9-9872-49ce0f86e611",
      "slug": "np-donation",
      "labels": {"singular": "Donation", "plural": "Donations"},
      "fields": [
        {"id": "amount", "type": "text"},
        {"id": "donation-date", "type": "datetime"},
        {"id": "campaign-name", "type": "text"}
      ],
      "relationships": [
        {
          "id": "primary-contact",
          "namespace": "contacts",
          "hasMany": false
        }
      ]
    }
  ]
}
```

## Get a single schema with full field details

```
GET /api/3/customObjects/schemas/{schemaId}?showFields=all
```

The `showFields=all` query parameter returns inherited fields and any marked-for-deletion fields. Without it the response only includes active first-class fields.

### Use the response to discover

- **`fields[].id`** — the field identifiers you'll use in record bodies.
- **`fields[].type`** — `text`, `datetime`, `number`, `boolean`, etc.
- **`relationships[].id`** — almost always `primary-contact` for contact-linked objects.
- **`labels.singular`** / **`slug`** — useful for finding a schema by name in code.

## Looking up by slug or label

The list endpoint returns all schemas in one page (typically <100 per account). The pragmatic pattern is to fetch the list once on startup and match locally:

```python
schemas = client.get("/customObjects/schemas?limit=100")["schemas"]
donation = next(s for s in schemas if s["labels"]["singular"] == "Donation")
schema_id = donation["id"]
```

> **Schema ID vs slug**
>
> Most record-level endpoints take the schema **UUID**, not the slug. Look up the UUID once and cache it for the session.

---

Next: [Step 2 — Records](../02-records)
