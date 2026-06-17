---
title: "Step 3 — Apply tags"
layout: default
parent: "Contacts, Lists & Tags"
nav_order: 3
permalink: /contacts-lists-tags/03-apply-tags/
---

# Step 3 — Apply tags

Two-step pattern: ensure the tag exists (create it if needed), then link it to the contact via the `contactTags` endpoint.

## Step 3a — Create (or get) a tag

If the tag already exists, skip to [Step 3b](#step-3b--apply-tag-to-contact) and use its numeric `id`. If you need to create it first:

```
POST https://{youraccountname}.api-us1.com/api/3/tags
```

```json
{
  "tag": {
    "tag": "Customer",
    "tagType": "contact",
    "description": "Paying customer"
  }
}
```

### `tag` fields

| Field | Required | Type | Notes |
|---|---|---|---|
| `tag` | **yes** | string | The tag name. Must be unique within the account. |
| `tagType` | **yes** | string | Must be `"contact"` for contact tags. Tags without this field are created but do not appear in the standard contact-tag UI. |
| `description` | no | string | Optional description shown in the AC UI. |

### Response

HTTP `201` on creation. The response wraps the new tag under a `tag` key:

```json
{
  "tag": {
    "tag": "Customer",
    "description": "Paying customer",
    "tagType": "contact",
    "cdate": "2026-06-17T06:46:20-05:00",
    "links": {
      "contactGoalTags": "https://{account}.api-us1.com/api/3/tags/471/contactGoalTags",
      "templateTags": "https://{account}.api-us1.com/api/3/tags/471/templateTags"
    },
    "id": "471"
  }
}
```

Capture `tag.id` — you need this numeric value in Step 3b.

To search for an existing tag by name, use `GET /api/3/tags?search={name}` — the `search` query parameter performs a substring match on the tag name. See the [official API reference for listing and searching tags](https://developers.activecampaign.com/reference/retrieve-all-tags) for the full parameter list. Read `tags[0].id` from the response.

---

## Step 3b — Apply tag to contact

```
POST https://{youraccountname}.api-us1.com/api/3/contactTags
```

```json
{
  "contactTag": {
    "contact": "1182",
    "tag": "471"
  }
}
```

### `contactTag` fields

| Field | Required | Type | Notes |
|---|---|---|---|
| `contact` | **yes** | string | Numeric ID of the contact. Get this from the `contact/sync` response in Step 1 or from `GET /api/3/contacts`. |
| `tag` | **yes** | string | Numeric ID of the tag. Get this from the `POST /api/3/tags` response or from `GET /api/3/tags`. Tag names are **not** accepted — passing a name string returns HTTP 422 `"No tag id provided"`. |

### Response

HTTP `200` on success. The response contains two top-level keys:

- **`contacts`** — an array with the full contact object for the tagged contact.
- **`contactTag`** — the contact-tag link record that was created.

Trimmed example:

```json
{
  "contacts": [
    {
      "id": "1182",
      "email": "test@example.com",
      "firstName": "Test",
      "lastName": "Contact",
      "cdate": "2026-05-27T02:20:03-05:00",
      "udate": "2026-06-17T06:46:26-05:00",
      "links": { "contactTags": "https://{account}.api-us1.com/api/3/contacts/1182/contactTags" }
    }
  ],
  "contactTag": {
    "contact": "1182",
    "tag": "471",
    "cdate": "2026-06-17T06:46:26-05:00",
    "links": {
      "tag": "https://{account}.api-us1.com/api/3/contactTags/4544/tag",
      "contact": "https://{account}.api-us1.com/api/3/contactTags/4544/contact"
    },
    "id": "4544"
  }
}
```

For the full response schema see the [official API reference](https://developers.activecampaign.com/reference/create-contact-tag).

> **`contactTag.id`** is the ID of the contact-tag link record, not the contact or tag. Use this ID to remove the tag from the contact later via `DELETE /api/3/contactTags/{id}` — this only removes the link between the contact and the tag; it does not delete the tag itself from your account.

## Key behaviors verified live

- **Tag must be referenced by numeric ID, not name.** Passing the tag name string (e.g. `"Customer"`) in the `tag` field returns HTTP 422 `"No tag id provided"`. Always resolve the tag to its numeric `id` first.
- **Applying the same tag twice is idempotent.** Re-posting the same `contact`/`tag` pair does not create a duplicate. The API returns HTTP `200` with the same `contactTag.id` and the original `cdate` unchanged. No error is raised.
- **`tagType: "contact"` is required.** Tags created without `tagType` are accepted by the API but do not appear in the standard contact-tag UI or in `GET /api/3/tags` results. Always include it.

## Pitfalls

- **Using the tag name instead of the numeric ID.** The `contactTags` endpoint requires `tag` to be the numeric ID string. Resolve tag names via `GET /api/3/tags?search={name}` before applying.
- **Duplicate tag names are not created.** `POST /api/3/tags` with a name that already exists returns HTTP `200` and the existing tag — it does not create a second tag. Always check for an existing tag first (via `GET /api/3/tags?search={name}`) and reuse its ID.

---

Next: [Step 4 — Set custom field values](../04-custom-fields)
