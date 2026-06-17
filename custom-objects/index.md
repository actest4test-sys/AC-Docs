---
title: Custom Objects
layout: default
nav_order: 3
has_children: true
permalink: /custom-objects/
---

# Custom Objects

**Schemas → Records → Querying**

Custom objects let you model any business entity in ActiveCampaign that isn't a contact, deal, or list — donations, course enrolments, event tickets, subscriptions, property viewings, anything domain-specific. Each custom object has a **schema** (the shape) and **records** (the instances), and records almost always link to a **contact** via a relationship.

This category covers the three core API tasks: discovering schemas, creating records, and querying them.

---

## Concepts

| Term | What it is |
|---|---|
| **Schema** | The blueprint — defines the object name, fields, and which other AC objects it relates to (typically a contact). |
| **Slug** | A URL-safe identifier for the schema (e.g. `np-donation`). |
| **Schema ID** | A UUID. **Most record-level endpoints take the schema ID, not the slug.** |
| **Field** | A typed attribute on the schema (`text`, `datetime`, `number`, etc.). Each field has its own `id` (e.g. `amount`). |
| **Record** | An instance of a schema (one donation, one ticket). Has its own UUID. |
| **Relationship** | A link from a record to a related object — almost always a contact via `primary-contact`. |
| **External ID** | Your system's ID for the record. Optional but useful for idempotent re-syncs. |

## The flow at a glance

1. **[Step 1 — Schemas](./01-schemas)** — list available schemas and inspect their fields & relationships.
2. **[Step 2 — Records](./02-records)** — create records linked to contacts.
3. **[Step 3 — Querying](./03-querying)** — retrieve records, filter by contact, paginate.

---

## Reference links

- [Custom Objects (developer docs)](https://developers.activecampaign.com/reference/custom-objects)
- [Custom Object schemas (developer docs)](https://developers.activecampaign.com/reference/custom-object-schemas)
- [Custom Objects explained (developer docs)](https://developers.activecampaign.com/docs/custom-objects-explained)
- [Custom Objects overview (AC help)](https://help.activecampaign.com/hc/en-us/articles/4408415902738-Custom-objects-overview)
