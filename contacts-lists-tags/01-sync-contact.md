---
title: "Step 1 — Sync a contact"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 1
permalink: /contacts-lists-tags/01-sync-contact/
---

# Step 1 — Sync a contact

Create-or-update a contact by email. Preferred over plain `POST /contacts` when you don't know whether the contact already exists.

```
POST /api/3/contact/sync
```

## Request body

*TBD — fill in the contact fields you actually pass from your integration.*

```json
{
  "contact": {
    "email": "TBD",
    "firstName": "TBD",
    "lastName": "TBD",
    "phone": "TBD"
  }
}
```

## Key fields

- *TBD*

## Response

*TBD — note the `contact.id` you get back; you'll need it for steps 2–4.*

## Pitfalls

- *TBD*

---

Next: [Step 2 — Subscribe to a list](../02-subscribe-list)
