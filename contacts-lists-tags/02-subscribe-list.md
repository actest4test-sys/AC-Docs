---
title: "Step 2 — Subscribe to a list"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 2
permalink: /contacts-lists-tags/02-subscribe-list/
---

# Step 2 — Subscribe to a list

Adds an existing contact to an existing list with a given subscription status.

```
POST /api/3/contactLists
```

## Request body

*TBD*

```json
{
  "contactList": {
    "list": "TBD",
    "contact": "TBD",
    "status": 1
  }
}
```

## Key fields

- *TBD — explain `status` values (1 = subscribed, 2 = unsubscribed, etc.).*

## Response

*TBD*

## Pitfalls

- *TBD — "already on list" duplicate-error pattern.*

---

Next: [Step 3 — Apply tags](../03-apply-tags)
