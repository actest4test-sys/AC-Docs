---
title: "Step 3 — Apply tags"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 3
permalink: /contacts-lists-tags/03-apply-tags/
---

# Step 3 — Apply tags

Two-step pattern: ensure the tag exists, then link it to the contact.

## Create (or get) a tag

```
POST /api/3/tags
```

*TBD*

```json
{
  "tag": {
    "tag": "TBD",
    "tagType": "contact"
  }
}
```

## Apply tag to contact

```
POST /api/3/contactTags
```

*TBD*

```json
{
  "contactTag": {
    "contact": "TBD",
    "tag": "TBD"
  }
}
```

## Key fields

- *TBD*

## Pitfalls

- *TBD — duplicate-tag-on-contact handling; tags without `tagType` not appearing in GET /tags.*

---

Next: [Step 4 — Set custom field values](../04-custom-fields)
