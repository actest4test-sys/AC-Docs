---
title: "Step 5 — Look up existing contacts"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 5
permalink: /contacts-lists-tags/05-lookup-contacts/
---

# Step 5 — Look up existing contacts

Use the contacts list endpoint to search for, filter, and retrieve contacts without performing an upsert. This is useful when you need to confirm a contact exists, retrieve their ID, or pull their current field values.

```
GET https://{youraccountname}.api-us1.com/api/3/contacts
```

Full reference: [List, search and filter contacts](https://developers.activecampaign.com/reference/list-all-contacts)

---

## Look up by email (exact match)

```
GET https://{youraccountname}.api-us1.com/api/3/contacts?email=jane@example.com
```

Returns a `contacts` array. If the contact exists, `contacts[0]` is your record. If not, `meta.total` is `0`.

```json
{
  "contacts": [
    {
      "id": "242",
      "email": "jane@example.com",
      "firstName": "Jane",
      "lastName": "Doe",
      "phone": "",
      "cdate": "2022-11-02T07:03:03-05:00",
      "udate": "2026-05-21T10:26:06-05:00"
    }
  ],
  "scoreValues": [],
  "meta": { "total": "1" }
}
```

> **Note:** `GET /contacts?email=...` returns `{contacts: [...]}` (plural array), whereas `GET /contacts/{id}` returns `{contact: {...}}` (singular object). Don't mix up the two shapes.

## Look up by ID

If you already have the contact's numeric ID:

```
GET https://{youraccountname}.api-us1.com/api/3/contacts/242
```

Returns the full contact object plus related collections (`contactAutomations`, `contactLists`, `contactTags`, `deals`, etc.).

## Filtering and pagination

Common query parameters for `GET /api/3/contacts`:

| Parameter | Description |
|---|---|
| `email` | Exact email match |
| `search` | Full-text search across name, email, org |
| `listid` | Filter by list membership (numeric list ID) |
| `tagid` | Filter by tag (numeric tag ID) |
| `status` | Subscription status (`1` = active, `2` = unsubscribed) |
| `limit` | Max results per page (default 20, max 100) |
| `offset` | Pagination offset |

Use `offset` to paginate: `?limit=100&offset=100` for the second page, and so on.

More filters and query parameters are available — see the [official reference](https://developers.activecampaign.com/reference/list-all-contacts).

## Pitfalls

- **`meta.total` is a string, not a number.** Compare with `=== "0"` or parse it — `meta.total === 0` will never be true.
- **`scoreValues` in the response is normal.** It's a sibling collection, not part of the contact object.
- **`GET /contacts/{id}` is heavier than it looks.** It returns the contact plus all related collections. For bulk lookups, use the list endpoint with filters instead.
- **You cannot filter contacts by custom field values via this endpoint.** The `GET /api/3/contacts` endpoint does not support filtering on `fieldValues`. If you need to find contacts by a custom field value, export contacts and filter client-side, or use segmentation/automations within the AC UI.

---

Back to: [Contacts, Lists & Tags overview](../)
