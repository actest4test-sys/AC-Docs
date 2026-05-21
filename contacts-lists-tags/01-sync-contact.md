---
title: "Step 1 — Sync a contact"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 1
has_children: true
permalink: /contacts-lists-tags/01-sync-contact/
---

# Step 1 — Sync a contact

Create-or-update a contact by email in a single call. **This is the endpoint you almost always want** when an integration receives data from an external system — it's idempotent and safe to re-run.

```
POST /api/3/contact/sync
```

## Request body

The minimum payload is just an email — every other field is optional and only updated if you send it.

```json
{
  "contact": {
    "email": "jane@example.com",
    "firstName": "Jane",
    "lastName": "Doe",
    "phone": "+353 87 123 4567"
  }
}
```

### Top-level `contact` fields

| Field | Required | Notes |
|---|---|---|
| `email` | **yes** | Used as the dedupe key. Missing email returns `400`; invalid email returns `422`. |
| `firstName` | no | |
| `lastName` | no | |
| `phone` | no | Free-form; AC does not validate format. |
| `fieldValues` | no | Array of `{field, value}` to set custom field values — see below. |

> **`sync` preserves fields you don't send.** A re-sync with only `{email, firstName}` does **not** clear `lastName` or `phone`. Only the keys present in the body are written.

## Response

`HTTP 201` (newly created) or `HTTP 200` (existing contact updated). Body shape:

```json
{
  "contact": {
    "id": "242",
    "email": "jane@example.com",
    "firstName": "Jane",
    "lastName": "Doe",
    "phone": "",
    "cdate": "2022-11-02T07:03:03-05:00",
    "udate": "2026-05-21T10:26:06-05:00",
    "bounced_hard": "0",
    "bounced_soft": "0",
    "sentcnt": "658",
    "last_open_date": "2023-02-02 01:12:22",
    "last_click_date": "2023-02-02 01:12:22",
    "orgid": "3",
    "orgname": "Dublin Museum",
    "deleted": "0",
    "links": { /* 18 related-resource URLs */ }
  }
}
```

**Always capture `contact.id`** — you'll need it for list subscriptions ([Step 2](../02-subscribe-list)), tag application ([Step 3](../03-apply-tags)), custom field writes ([Step 4](../04-custom-fields)), custom object relationships ([Custom Objects](../../custom-objects/02-records/)), and ecommerce customer linkage ([Ecommerce](../../ecommerce/02-customer/)).

The `links` object contains URLs to related resources (`contactAutomations`, `contactLists`, `contactTags`, `fieldValues`, `deals`, etc.) — handy for follow-up calls without constructing URLs yourself.

## Setting custom field values during sync

Custom contact fields can be set in the same call by including a `fieldValues` array:

```json
{
  "contact": {
    "email": "jane@example.com",
    "firstName": "Jane",
    "fieldValues": [
      {"field": "1",  "value": "Premium"},
      {"field": "5",  "value": "Kildare"},
      {"field": "14", "value": "Opportunity"}
    ]
  }
}
```

- **`field`** is the **numeric ID** from `GET /api/3/fields`, **not** the `perstag`.
- **`value`** is always a string (even for numeric/date fields).
- When `fieldValues` is in the request, the response includes both a `fields` key and a `fieldValues` array listing **all** field values on the contact (not just the ones you set).

See [Step 4](../04-custom-fields) for the standalone `POST /fieldValues` pattern (useful when you only need to update one custom field after the contact already exists).

## Errors

### Missing email — `HTTP 400`

```json
{"errors":[{"status":400,"title":"Bad Request","detail":"Contact email must be provided for sync"}]}
```

### Invalid email — `HTTP 422`

```json
{
  "errors": [{
    "title": "Contact Email Address is not valid.",
    "code": "email_invalid",
    "error": "must_be_valid_email_address",
    "source": {"pointer": "/data/attributes/email"}
  }]
}
```

The two shapes differ: `400` errors are flat (`{status, title, detail}`), `422` errors are validation-style (`{title, code, error, source}`). Handle both.

## `POST /contact/sync` vs `POST /contacts`

| | `/contact/sync` | `/contacts` |
|---|---|---|
| Behavior | Upsert by email | Create only |
| Existing email | Returns the existing contact (updates if data provided) | `HTTP 422` with `code: "duplicate"` |
| Use when | Integrating from any external system | You specifically need create-only semantics and your own duplicate handling |

Sample duplicate error from `POST /contacts`:

```json
{
  "errors": [{
    "title": "Email address already exists in the system.",
    "code": "duplicate",
    "source": {"pointer": "/data/attributes/email"}
  }]
}
```

In almost every integration scenario, prefer `/contact/sync`.

> **For high-volume / historical loads, use the bulk import endpoint instead.**
>
> `/contact/sync` is fine for real-time, one-at-a-time integration. For batch backfills or large imports, see [Bulk import](./bulk-import) — it accepts up to 250 contacts per request and is asynchronous.

## Looking up an existing contact

If you already have the contact ID, use:

```
GET /api/3/contacts/{id}
```

Returns the contact plus related collections (automations, lists, deals, etc.) — a heavy response, fine for one-off lookups.

To find a contact by email without doing an upsert:

```
GET /api/3/contacts?email=jane@example.com
```

Returns `{contacts: [...], scoreValues: [...], meta: {total, ...}}`. The `scoreValues` key sitting next to `contacts` is normal — it's not the response body, it's a sibling collection. Read `contacts[0]` (or `meta.total === 0` for "not found").

`email_like=` accepts a substring instead of an exact match.

## Pitfalls

- **Sending `field` as a `perstag` instead of the numeric ID.** Use `GET /api/3/fields` first to resolve. Wrong `field` values are silently ignored.
- **Assuming `sync` clears unspecified fields.** It doesn't — re-sync only touches keys you send.
- **Using `POST /contacts` for integrations.** It throws 422 on every re-sync of an existing email. Use `/contact/sync`.
- **Reading the response as `{contacts: [...]}` after a single-contact lookup.** `GET /contacts/{id}` returns `{contact: {...}}` (singular). `GET /contacts?email=...` returns `{contacts: [...]}` (plural).
- **Ignoring `bounced_hard`.** A contact with `bounced_hard: "1"` won't receive email. Worth surfacing in your sync logic.

---

Next: [Step 2 — Subscribe to a list](../02-subscribe-list)
