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

```bash
curl -X POST \
  -H "Api-Token: {yourapikey}" \
  -H "Content-Type: application/json" \
  "https://{youraccountname}.api-us1.com/api/3/contactLists" \
  -d '{
    "contactList": {
      "list": "8",
      "contact": "1182",
      "status": 1
    }
  }'
```

### `contactList` fields

| Field | Required | Type | Notes |
|---|---|---|---|
| `list` | **yes** | string | Numeric ID of the list. Get list IDs from `GET /api/3/lists`. |
| `contact` | **yes** | string | Numeric ID of the contact. Get this from the `contact/sync` response or `GET /api/3/contacts`. |
| `status` | **yes** | integer | Subscription status — see table below. |
| `sourceid` | no | integer | Source of the subscription. `0` = unknown, `4` = API. Defaults to `4` when posting via the API. |

## Status values

All four values below are accepted by the API. Values outside `0–3` return HTTP 422.

| Value | Meaning |
|---|---|
| `1` | **Subscribed** — contact is active on the list and eligible to receive campaigns. Use this for the standard subscribe case. |
| `2` | **Unsubscribed** — contact has been removed from the list. AC records the unsubscribe date (`udate`) and the contact will not receive campaigns for this list. |
| `3` | **Unconfirmed (pending)** — contact has been added but has not confirmed their subscription. Used when a list requires double opt-in confirmation. The contact will not receive campaigns until they confirm. |
| `0` | **Not set** — the contact record on the list exists but has no active subscription state. Rarely used directly; AC may set this internally when a contact is removed without a formal unsubscribe event. |

> **In practice, almost all API integrations use `1` (subscribe) or `2` (unsubscribe).** Use `3` only if your list is configured for double opt-in and you are handling the confirmation flow yourself.

## Response

HTTP `200` on success. The response contains two top-level keys:

- **`contacts`** — an array with the full contact object for the contact whose list status was updated.
- **`contactList`** — the list membership record that was created or updated.

Trimmed example (status=1, subscribe):

```json
{
  "contacts": [
    {
      "id": "1182",
      "email": "test@example.com",
      "firstName": "Test",
      "lastName": "Contact",
      "cdate": "2026-05-27T02:20:03-05:00",
      "udate": "2026-06-17T06:35:11-05:00",
      "links": { "contactLists": "https://{account}.api-us1.com/api/3/contacts/1182/contactLists" }
    }
  ],
  "contactList": {
    "id": "628",
    "contact": 1182,
    "list": 8,
    "status": 1,
    "sdate": "2026-06-17T06:35:11-05:00",
    "udate": null,
    "unsubreason": "",
    "sourceid": 4,
    "campaign": null,
    "message": null,
    "unsubscribeAutomation": null,
    "links": {
      "list": "https://{account}.api-us1.com/api/3/contactLists/628/list",
      "contact": "https://{account}.api-us1.com/api/3/contactLists/628/contact",
      "automation": "https://{account}.api-us1.com/api/3/contactLists/628/automation"
    }
  }
}
```

For the full response schema see the [official API reference](https://developers.activecampaign.com/reference/update-list-status-for-contact).

> **`contactList.id`** is the ID of the list-membership record, not the contact or list. The `sdate` field is set when the contact first subscribes; re-posting with the same `list`/`contact` pair updates the record in place rather than creating a new one.

## Pitfalls

- **Sending `list` or `contact` as the name/email instead of the numeric ID.** The endpoint requires integer IDs. Use `GET /api/3/lists` to resolve list names to IDs, and capture `contact.id` from the `/contact/sync` response in Step 1.
- **Re-subscribing an unsubscribed contact.** Posting `status: 1` for a contact that previously had `status: 2` (unsubscribed) is valid — the API updates the record and resets the status. However, review your local compliance policy before programmatically re-subscribing contacts.
- **`status: 4` and above return `HTTP 422`.** The API rejects any value outside `0–3` with `"The value you selected is not a valid choice."` — guard against passing arbitrary integers from an upstream system.

---

Next: [Step 3 — Apply tags](../03-apply-tags)
