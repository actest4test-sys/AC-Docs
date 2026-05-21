---
title: ActiveCampaign Ecommerce API
description: Connection → Customer → Order → Abandoned Cart — the four core REST calls for pushing ecommerce data into ActiveCampaign.
---

# ActiveCampaign Ecommerce API

**Connection → Customer → Order → Abandoned Cart**

This guide walks through the four core REST API calls needed to push ecommerce data from a custom integration into ActiveCampaign: creating a store connection, attaching a customer, creating an order, and creating an abandoned cart. Use it alongside the official [ActiveCampaign Developer Docs](https://developers.activecampaign.com/).

---

## Before you start

### Authentication

All requests authenticate via an API key sent in the `Api-Token` header. You can find your key under **Settings → Developer** in the AC UI. Full details: [Authentication](https://developers.activecampaign.com/reference/authentication).

### Base URL

Every endpoint sits under your account-specific base URL:

```
https://{youraccountname}.api-us1.com/api/3/
```

Replace `{youraccountname}` with the subdomain in the AC URL bar. Reference: [Base URL](https://developers.activecampaign.com/reference/url).

### Standard headers

```
Api-Token: YOUR_API_KEY
Content-Type: application/json
Accept: application/json
```

---

## The flow at a glance

1. **Create a connection** — represents the external ecommerce store.
2. **Create a customer** — links a contact in AC to a customer record in that store.
3. **Create an order** — a completed purchase tied to the customer and connection.
4. **Create an abandoned cart** — same endpoint as orders, but with cart-specific fields. Update later when (or if) the cart converts.

---

## Step 1 — Create a connection (the store)

A connection is the AC-side representation of an external ecommerce store. Every customer and order must reference a `connectionid`. You usually create this once per store. Full reference: [Create a connection](https://developers.activecampaign.com/reference/create-connection).

```
POST /api/3/connections
```

**Request body**

```json
{
  "connection": {
    "service": "my-custom-store",
    "externalid": "store-12345",
    "name": "My Custom Store",
    "logoUrl": "https://example.com/logo.png",
    "linkUrl": "https://example.com"
  }
}
```

**Key fields**

- `service` — short identifier for the store/platform (lowercase, no spaces). This is what surfaces in the AC UI integration list.
- `externalid` — your unique ID for the store. Use this to look up the connection later.
- `name` — display name shown in AC.
- `logoUrl` / `linkUrl` — surfaced in the AC UI; optional but recommended.

**Response**

Returns the created connection including its `id` (numeric). Store this — every customer and order needs it as `connectionid`.

---

## Step 2 — Create an ecommerce customer

A customer record links an AC contact (by email) to a connection. Orders attach to customers, and the customer aggregate (lifetime revenue, order count, etc.) is calculated from this object. Full reference: [Create a customer](https://developers.activecampaign.com/reference/create-customer).

```
POST /api/3/ecomCustomers
```

**Request body**

```json
{
  "ecomCustomer": {
    "connectionid": "1",
    "externalid": "customer-789",
    "email": "jane@example.com",
    "acceptsMarketing": "1"
  }
}
```

**Key fields**

- `connectionid` — the `id` returned from Step 1.
- `externalid` — the customer's ID in your store. Must be unique per connection.
- `email` — required. AC will create a contact with this email if one doesn't already exist.
- `acceptsMarketing` — `"1"` or `"0"`. Reflects marketing consent in the source system.

> **⚠ If the customer already exists**
>
> Creating a customer with the same `connectionid` + `externalid` combo will return a `422` error. Either `GET` the customer first (filter on `externalid` and `connectionid`), or skip straight to creating the order — AC will auto-create a customer behind the scenes when you provide the same identifiers on the order.

---

## Step 3 — Create an order

Orders are the heart of the ecommerce integration. Creating one will (a) create the AC contact if needed, (b) attach the order to the customer record, and (c) fire any "Makes a purchase" automations. Full reference: [Create an order](https://developers.activecampaign.com/reference/create-an-order).

```
POST /api/3/ecomOrders
```

**Request body**

```json
{
  "ecomOrder": {
    "externalid": "order-1001",
    "source": 1,
    "email": "jane@example.com",
    "orderProducts": [
      {
        "externalid": "sku-001",
        "name": "Blue Widget",
        "price": 2500,
        "quantity": 2,
        "category": "Widgets",
        "sku": "BW-001",
        "description": "A blue widget",
        "imageUrl": "https://example.com/blue-widget.png",
        "productUrl": "https://example.com/products/blue-widget"
      }
    ],
    "orderUrl": "https://example.com/orders/1001",
    "externalCreatedDate": "2026-05-21T10:00:00-05:00",
    "shippingMethod": "Standard",
    "totalPrice": 5500,
    "shippingAmount": 500,
    "taxAmount": 0,
    "discountAmount": 0,
    "currency": "USD",
    "connectionid": "1",
    "customerid": "1"
  }
}
```

**Key fields**

- `externalid` — order ID in the source system. Must be unique per connection.
- `source` — **very important**. `1` = real-time (triggers automations). `0` = historical (silent — use for migrations).
- `email` — the customer's email. Used to match/create the contact.
- `totalPrice`, `shippingAmount`, `taxAmount`, `discountAmount` — all in **cents** (or the smallest currency unit). €25.00 = `2500`.
- `currency` — three-letter ISO code, e.g. `USD`, `EUR`, `GBP`.
- `externalCreatedDate` — ISO 8601 with timezone offset. This is the order date that drives reporting and automation timing.
- `connectionid` / `customerid` — link the order to the records from Steps 1 and 2.
- `orderProducts` — array of line items. Each product needs at minimum `externalid`, `name`, `price`, and `quantity`.

> **⚠ Prices are in the smallest currency unit**
>
> Pass `2500` for $25.00 / €25.00 / £25.00. Do not send decimals. This is the single most common source of bugs in custom AC ecommerce integrations.

---

## Step 4 — Create an abandoned cart

An abandoned cart uses the **same endpoint** as an order (`/ecomOrders`) — what makes it a cart rather than a completed order is the presence of two fields: `externalcheckoutid` and `abandonedDate`. Reference: [E-Commerce Abandoned Carts](https://developers.activecampaign.com/reference/abandoned-cart).

```
POST /api/3/ecomOrders
```

**Request body**

```json
{
  "ecomOrder": {
    "externalcheckoutid": "checkout-abc-555",
    "source": 1,
    "email": "jane@example.com",
    "abandonedDate": "2026-05-21T10:30:00-05:00",
    "orderProducts": [
      {
        "externalid": "sku-001",
        "name": "Blue Widget",
        "price": 2500,
        "quantity": 1,
        "category": "Widgets",
        "imageUrl": "https://example.com/blue-widget.png",
        "productUrl": "https://example.com/products/blue-widget"
      }
    ],
    "orderUrl": "https://example.com/checkout/abc-555",
    "externalCreatedDate": "2026-05-21T10:00:00-05:00",
    "totalPrice": 2500,
    "currency": "USD",
    "connectionid": "1",
    "customerid": "1"
  }
}
```

**Cart-specific fields**

- `externalcheckoutid` — the unique cart/checkout ID from your platform. This is the field that flags the record as a cart rather than an order.
- `abandonedDate` — the timestamp the cart was considered abandoned. This is the timestamp the "Abandons cart" automation trigger fires from.
- `orderUrl` — should be the return-to-checkout URL. AC's "Abandoned cart" content block uses this to populate the "Return to checkout" button.
- Notice there is **no `externalid`** — leaving this off is what keeps the record in the cart state.

### Converting a cart into an order

When the customer comes back and completes the purchase, **update** (don't create a new) record by `PUT`ting to `/api/3/ecomOrders/{id}` and adding the `externalid` field. AC transitions the record from cart → order, removes the contact from the "Abandons cart" trigger, and fires "Makes a purchase" instead.

```
PUT /api/3/ecomOrders/{id}
```

```json
{
  "ecomOrder": {
    "externalid": "order-1001",
    "totalPrice": 2500
  }
}
```

> **⚠ Don't write directly to this API if you're using Shopify, WooCommerce, BigCommerce, or Magento**
>
> If a native AC ecommerce integration is already installed for the platform, don't push data through this REST API in parallel — you'll end up with duplicate records and out-of-sync data. The REST endpoints in this guide are for **custom stores with no native AC connector**.

---

## Quick recap

| Step | Endpoint | What makes it unique |
|---|---|---|
| Connection | `POST /api/3/connections` | Created once per store. Returns the `connectionid` used everywhere. |
| Customer | `POST /api/3/ecomCustomers` | Links email → connection. Auto-created if you skip and just send orders. |
| Order | `POST /api/3/ecomOrders` | Has `externalid`. `source: 1` triggers automations. |
| Abandoned cart | `POST /api/3/ecomOrders` | Has `externalcheckoutid` + `abandonedDate`, no `externalid`. |
| Cart → order | `PUT /api/3/ecomOrders/{id}` | Add `externalid` to the cart record to convert it. |

---

## Reference links

- [API overview](https://developers.activecampaign.com/reference/overview)
- [Authentication](https://developers.activecampaign.com/reference/authentication)
- [Rate limits](https://developers.activecampaign.com/reference/rate-limits)
- [DeepData — Connections](https://developers.activecampaign.com/reference/connection)
- [DeepData — Ecommerce Customers](https://developers.activecampaign.com/reference/customer)
- [DeepData — Ecommerce Orders](https://developers.activecampaign.com/reference/orders)
- [DeepData — Abandoned Carts](https://developers.activecampaign.com/reference/abandoned-cart)
- [Abandoned cart automation (AC help center)](https://help.activecampaign.com/hc/en-us/articles/220500087-Abandoned-Cart-Automation)
