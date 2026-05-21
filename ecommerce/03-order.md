---
title: "Step 3 — Order"
layout: default
parent: Ecommerce
nav_order: 3
permalink: /ecommerce/03-order/
---

# Step 3 — Create an order

Orders are the heart of the ecommerce integration. Creating one will (a) create the AC contact if needed, (b) attach the order to the customer record, and (c) fire any "Makes a purchase" automations. Full reference: [Create an order](https://developers.activecampaign.com/reference/create-order).

```
POST /api/3/ecomOrders
```

## Request body

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

## Key fields

- `externalid` — order ID in the source system. Must be unique per connection.
- `source` — **very important**. `1` = real-time (triggers automations). `0` = historical (silent — use for migrations).
- `email` — the customer's email. Used to match/create the contact.
- `totalPrice`, `shippingAmount`, `taxAmount`, `discountAmount` — all in **cents** (or the smallest currency unit). €25.00 = `2500`.
- `currency` — three-letter ISO code, e.g. `USD`, `EUR`, `GBP`.
- `externalCreatedDate` — ISO 8601 with timezone offset. This is the order date that drives reporting and automation timing.
- `connectionid` / `customerid` — link the order to the records from [Step 1](../01-connection) and [Step 2](../02-customer).
- `orderProducts` — array of line items. Each product needs at minimum `externalid`, `name`, `price`, and `quantity`.

> **⚠ Prices are in the smallest currency unit**
>
> Pass `2500` for $25.00 / €25.00 / £25.00. Do not send decimals. This is the single most common source of bugs in custom AC ecommerce integrations.

---

Next: [Step 4 — Create an abandoned cart](../04-abandoned-cart)
