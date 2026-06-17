---
title: "Contacts, Lists & Tags"
layout: default
nav_order: 4
has_children: true
has_toc: false
permalink: /contacts-lists-tags/
---

# Contacts, Lists & Tags

**Sync → Subscribe → Tag → Custom fields**

The four most common contact-shaping operations from a custom integration.

---

## Before you start

Authentication and base URL follow the same pattern as [Ecommerce](../ecommerce/) — `Api-Token` header against `https://{youraccountname}.api-us1.com/api/3/`.

## The flow at a glance

1. **Step 1 — [Sync contacts individually](./01-sync-contact) or [Import in Bulk](./01-sync-contact/bulk-import)**
2. **[Step 2 — Subscribe to a list](./02-subscribe-list)** — add a contact to a list with the right status.
3. **[Step 3 — Apply tags](./03-apply-tags)** — tag for segmentation and automation triggers.
4. **[Step 4 — Set custom field values](./04-custom-fields)** — write to custom contact fields.
5. **[Step 5 — Look up existing contacts](./05-lookup-contacts)** — search, filter, and retrieve contacts by email or other criteria.

> **Scaffolded section** — endpoint paths are set, body and key-field detail is TBD. Fill in as you write.

---

## Reference links

- [API overview](https://developers.activecampaign.com/reference/overview)
- [Authentication](https://developers.activecampaign.com/reference/authentication)
- [Sync a contact (developer docs)](https://developers.activecampaign.com/reference/sync-a-contacts-data)
- [List, search and filter contacts (developer docs)](https://developers.activecampaign.com/reference/list-all-contacts)
- [Retrieve fields (developer docs)](https://developers.activecampaign.com/reference/retrieve-fields)
- [Contact lists — subscribe/unsubscribe (developer docs)](https://developers.activecampaign.com/reference/update-list-status-for-contact)
- [Tags (developer docs)](https://developers.activecampaign.com/reference/create-a-new-tag)
- [Field values (developer docs)](https://developers.activecampaign.com/reference/create-fieldvalue)

---

## Table of contents
{: .text-delta }

- [Step 1 — Sync a contact](./01-sync-contact)
  - [Bulk import](./01-sync-contact/bulk-import)
- [Step 2 — Subscribe to a list](./02-subscribe-list)
- [Step 3 — Apply tags](./03-apply-tags)
- [Step 4 — Set custom field values](./04-custom-fields)
- [Step 5 — Look up existing contacts](./05-lookup-contacts)
