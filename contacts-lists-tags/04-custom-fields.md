---
title: "Step 4 — Set custom field values"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 4
permalink: /contacts-lists-tags/04-custom-fields/
---

# Step 4 — Set custom field values

Custom fields are defined separately from records — write values via the `fieldValues` endpoint, referencing the field's numeric ID and the contact's ID.

```
POST /api/3/fieldValues
```

## Request body

*TBD*

```json
{
  "fieldValue": {
    "contact": "TBD",
    "field": "TBD",
    "value": "TBD"
  }
}
```

## Key fields

- *TBD — `field` is the numeric ID from `GET /fields`, not the perstag.*

## Listing field definitions

```
GET /api/3/fields
```

*TBD — describe response shape and how to find a field by `perstag`.*

## Pitfalls

- *TBD — datetime / dropdown / multiselect formatting differences.*

---

Back to: [Contacts, Lists & Tags overview](../)
