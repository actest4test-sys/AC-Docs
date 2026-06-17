---
title: "Step 4 — Set custom field values"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 4
permalink: /contacts-lists-tags/04-custom-fields/
---

# Step 4 — Set custom field values

Custom fields are defined at the account level. Write values for a specific contact via the `fieldValues` endpoint, referencing the field's numeric ID (not its `perstag` name) and the contact's numeric ID.

## Request

```
POST https://{youraccountname}.api-us1.com/api/3/fieldValues
Api-Token: {yourapikey}
Content-Type: application/json
```

```json
{
  "fieldValue": {
    "contact": "1182",
    "field": "9",
    "value": "Acme Corp"
  }
}
```

### `fieldValue` fields

| Field | Required | Type | Notes |
|---|---|---|---|
| `contact` | **yes** | string | Numeric ID of the contact. Get this from the `contact/sync` response in Step 1 or from `GET /api/3/contacts`. |
| `field` | **yes** | string | Numeric ID of the custom field. Get field IDs from `GET /api/3/fields`. The `perstag` string (e.g. `"ACCOUNT"`) is **not** accepted — passing it returns HTTP 422 `"Field id not valid"`. |
| `value` | **yes** | string | The value to set. Format depends on field type — see the table below. |

## Response

HTTP `200` on success. The response contains two top-level keys:

- **`contacts`** — an array with the full contact object for the updated contact.
- **`fieldValue`** — the field value record that was created or updated.

Trimmed example (text field):

```json
{
  "contacts": [
    {
      "id": "1182",
      "email": "test@example.com",
      "firstName": "Test",
      "lastName": "Contact",
      "cdate": "2026-05-27T02:20:03-05:00",
      "udate": "2026-06-17T06:47:13-05:00",
      "links": { "fieldValues": "https://{account}.api-us1.com/api/3/contacts/1182/fieldValues" }
    }
  ],
  "fieldValue": {
    "contact": "1182",
    "field": "9",
    "value": "Acme Corp",
    "cdate": "2026-06-17T06:47:13-05:00",
    "udate": "2026-06-17T06:47:13-05:00",
    "links": {
      "owner": "https://{account}.api-us1.com/api/3/fieldValues/5725/owner",
      "field": "https://{account}.api-us1.com/api/3/fieldValues/5725/field"
    },
    "owner": "1182",
    "id": "5725"
  }
}
```

For the full response schema see the [official API reference](https://developers.activecampaign.com/reference/create-fieldvalue-1).

> **`fieldValue.id`** is the ID of the value record, not the contact or field definition. The `cdate` is set on first write; `udate` is updated on each subsequent write. The `owner` field echoes the contact's numeric ID.

## Field types and value format

| Field type | `value` format | Example |
|---|---|---|
| `text` | Plain string | `"Acme Corp"` |
| `textarea` | Plain string (newlines allowed) | `"Line 1\nLine 2"` |
| `hidden` | Plain string | `"internal-ref-001"` |
| `date` | `YYYY-MM-DD` | `"2023-01-15"` |
| `datetime` | ISO 8601 datetime string | `"2023-01-15T09:00:00-05:00"` |
| `radio` | The option's text value | `"Option A"` |
| `dropdown` | The option's text value | `"Lead"` |
| `checkbox` / `listbox` | Pipe-delimited string: `\|\|value1\|\|value2\|\|` | `"||Newsletter||Client||"` |

### Multi-value fields (checkbox / listbox)

Fields that allow multiple selections (type `checkbox` or `listbox`) use a pipe-delimited format with a leading and trailing `||`:

```json
{
  "fieldValue": {
    "contact": "1182",
    "field": "33",
    "value": "||Newsletter||Client||"
  }
}
```

The values must match the option text exactly as defined in the field — use [`GET /api/3/fields/{id}/options`](https://developers.activecampaign.com/reference/retrieve-fields) to retrieve the valid option values for a field.

## Pitfalls

- **Using `perstag` instead of the numeric `field` ID.** The endpoint always requires the numeric ID. Build a lookup map at startup using `GET /api/3/fields` if your integration references fields by name.
- **Wrong format for dropdown/radio.** These fields expect the option's text value string (e.g. `"Lead"`), not the option's numeric ID. Use `GET /api/3/fields/{id}/options` to confirm the exact text values before writing.
- **Missing `||` delimiters for checkbox/listbox.** Without the leading and trailing pipes, some checkbox implementations treat the entire string as a single selection. Always wrap multi-value strings in `||…||`.

---

Back to: [Contacts, Lists & Tags overview](../)
