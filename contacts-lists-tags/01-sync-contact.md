---
title: "Step 1 — Sync a contact"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 1
has_children: true
permalink: /contacts-lists-tags/01-sync-contact/
---

# Step 1 — Sync a contact

> **Tip — get your custom field IDs first.** If you plan to set custom field values during sync (via `fieldValues`), call the [Retrieve Fields endpoint](https://developers.activecampaign.com/reference/retrieve-fields) first to get the numeric IDs for each custom field. You can store those IDs on your side to avoid repeating the lookup, or simply run the call at the start of each sync and reference the IDs as needed.

Create-or-update a contact by email in a single call. **This is the endpoint you almost always want** when an integration receives data from an external system — it's idempotent and safe to re-run.

```
POST /api/3/contact/sync
```

## Request body

The minimum payload is just an email — every other field is optional and only updated if you send it.

```bash
curl -X POST \
  -H "Api-Token: {yourapikey}" \
  -H "Content-Type: application/json" \
  "https://{youraccountname}.api-us1.com/api/3/contact/sync" \
  -d '{
    "contact": {
      "email": "jane@example.com",
      "firstName": "Jane",
      "lastName": "Doe",
      "phone": "+353 87 123 4567"
    }
  }'
```

### Top-level `contact` fields

| Field | Required | Notes |
|---|---|---|
| `email` | **yes** | Used as the dedupe key. Missing email returns `400`; invalid email returns `422`. |
| `firstName` | no | Contact's first (given) name. |
| `lastName` | no | Contact's last (family) name. |
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
    "links": { "...": "18 related-resource URLs" }
  }
}
```

For the full response shape, see the [official API reference](https://developers.activecampaign.com/reference/sync-a-contacts-data).

**Always capture `contact.id`** — you'll need it for list subscriptions ([Step 2](../02-subscribe-list)), tag application ([Step 3](../03-apply-tags)), custom field writes ([Step 4](../04-custom-fields)), custom object relationships ([Custom Objects](../../custom-objects/02-records/)), and ecommerce customer linkage ([Ecommerce](../../ecommerce/02-customer/)). You can also obtain a contact's ID at any time by using the [List, search and filter contacts](../05-lookup-contacts) endpoint if you don't have it on hand.

> **Tip** — The `links` object contains URLs to related resources (`contactAutomations`, `contactLists`, `contactTags`, `fieldValues`, `deals`, etc.) — handy for follow-up calls without constructing URLs yourself.

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
- When `fieldValues` is in the request, the response includes a `fieldValues` array listing **only the field values you set** in that call (not all field values on the contact).

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

## `POST /contact/sync` vs `PUT /contacts/{id}`

| | `/contact/sync` | `/contacts/{id}` |
|---|---|---|
| Behavior | Upsert by email | Update by contact ID |
| Existing email | Returns the existing contact (updates if data provided) | `HTTP 422` with `code: "duplicate"` if you try to change email to one that already exists |
| Use when | Integrating from any external system | You specifically need update-only semantics and already have the contact ID |

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
> `/contact/sync` is fine for real-time, one-at-a-time integration. For batch backfills, large imports or updates affecting more than 10 contacts at once, see [Bulk import](./bulk-import) — it accepts up to 250 contacts per request and is asynchronous.

## Pitfalls

- **Sending `field` as a `perstag` instead of the numeric ID.** Use `GET /api/3/fields` first to resolve. Wrong `field` values are silently ignored.
- **Assuming `sync` clears unspecified fields.** It doesn't — re-sync only touches keys you send.
- **Using `POST /contacts` for integrations.** It throws 422 on every re-sync of an existing email. Use `/contact/sync`.

---

Next: [Step 2 — Subscribe to a list](../02-subscribe-list)
