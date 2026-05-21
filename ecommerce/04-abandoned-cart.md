---
title: "Step 4 — Abandoned Cart"
layout: default
parent: Ecommerce
nav_order: 4
permalink: /ecommerce/04-abandoned-cart/
---

# Step 4 — Create an abandoned cart

An abandoned cart uses the **same endpoint** as an order (`/ecomOrders`) — what makes it a cart rather than a completed order is the presence of two fields: `externalcheckoutid` and `abandonedDate`. Reference: [E-Commerce Abandoned Carts](https://developers.activecampaign.com/reference/abandoned-cart).

```
POST /api/3/ecomOrders
```

## Request body

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

## Cart-specific fields

- `externalcheckoutid` — the unique cart/checkout ID from your platform. This is the field that flags the record as a cart rather than an order.
- `abandonedDate` — the timestamp the cart was considered abandoned. This is the timestamp the "Abandons cart" automation trigger fires from.
- `orderUrl` — should be the return-to-checkout URL. AC's "Abandoned cart" content block uses this to populate the "Return to checkout" button.
- Notice there is **no `externalid`** — leaving this off is what keeps the record in the cart state.

## Converting a cart into an order

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

Back to: [Ecommerce overview](../)
