---
title: "Step 1 — Connection"
layout: default
parent: Ecommerce
nav_order: 1
permalink: /ecommerce/01-connection/
---

# Step 1 — Create a connection (the store)

A connection is the AC-side representation of an external ecommerce store. Every customer and order must reference a `connectionid`. You usually create this once per store. Full reference: [Create a connection](https://developers.activecampaign.com/reference/create-connection).

```
POST https://{youraccountname}.api-us1.com/api/3/connections
```

## Request body

```json
{
  "connection": {
    "service": "my-custom-store",
    "externalid": "store-12345",
    "name": "My Custom Store",
    "logoUrl": "https://example.com/logo.png",
    "linkUrl": "https://example.com",
    "listid": 1
  }
}
```

## Key fields

- `service` — short identifier for the store/platform (lowercase, no spaces). This is what surfaces in the AC UI integration list.
- `externalid` — your unique ID for the store. Use this to look up the connection later.
- `name` — display name shown in AC.
- `logoUrl` — image shown on the store/connection card in the AC UI.
- `linkUrl` — click-through destination for the store/connection card in the AC UI.
- `listid` — numeric ID of the AC list contacts coming through this connection should be added to.

## Response

Returns the created connection including its `id` (numeric). Store this — every customer and order needs it as `connectionid`.

---

Next: [Step 2 — Create a customer](../02-customer)
