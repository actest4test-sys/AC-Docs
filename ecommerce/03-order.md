---
title: "Step 3 — Order"
layout: default
parent: Ecommerce
nav_order: 3
permalink: /ecommerce/03-order/
---

# Step 3 — Create an order

Orders are the heart of the ecommerce integration. Creating one attaches the order to the customer record and fires any "Makes a purchase" automations. Full reference: [Create an order](https://developers.activecampaign.com/reference/create-order).

```
POST https://{youraccountname}.api-us1.com/api/3/ecomOrders
```

## Request body

```json
{
  "ecomOrder": {
    "externalid": "order-1001",
    "source": "1",
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
      },
      {
        "externalid": "sku-002",
        "name": "Red Widget",
        "price": 1500,
        "quantity": 1,
        "category": "Widgets",
        "sku": "RW-002",
        "description": "A red widget",
        "imageUrl": "https://example.com/red-widget.png",
        "productUrl": "https://example.com/products/red-widget"
      }
    ],
    "orderDiscounts": [
      {
        "name": "WELCOME10",
        "type": "order",
        "discountAmount": 500
      }
    ],
    "orderUrl": "https://example.com/orders/1001",
    "externalCreatedDate": "2026-05-21T10:00:00-05:00",
    "externalUpdatedDate": "2026-05-21T11:00:00-05:00",
    "shippingMethod": "Standard",
    "totalPrice": 6500,
    "shippingAmount": 500,
    "taxAmount": 0,
    "discountAmount": 500,
    "currency": "USD",
    "orderNumber": "myorder-1001",
    "connectionid": "1",
    "customerid": "1"
  }
}
```

## Key fields

- `externalid` — order ID in the source system. Must be unique per connection.
- `source` — **very important**. `"1"` = real-time (triggers automations). `"0"` = historical (silent — use for migrations).
- `email` — the customer's email. Used to match the contact in AC.
- `totalPrice`, `shippingAmount`, `taxAmount`, `discountAmount` — all in **cents** (or the smallest currency unit). €25.00 = `2500`.
- `currency` — three-letter ISO code, e.g. `USD`, `EUR`, `GBP`.
- `externalCreatedDate` / `externalUpdatedDate` — ISO 8601 with timezone offset. `externalCreatedDate` is the order date that drives reporting and automation timing; `externalUpdatedDate` is the last-modified date in the source system.
- `connectionid` / `customerid` — link the order to the records from [Step 1](../01-connection) and [Step 2](../02-customer).
- `orderProducts` — array of line items. Each product needs at minimum `externalid`, `name`, `price`, and `quantity`.
- `orderDiscounts` — array of discounts applied to the order. Each entry needs `name`, `type`, and `discountAmount` (in the smallest currency unit).
- `orderNumber` — human-facing order number shown to the customer in the source system. May differ from `externalid`.

> **⚠ Prices are in the smallest currency unit**
>
> Pass `2500` for $25.00 / €25.00 / £25.00. Do not send decimals. This is the single most common source of bugs in custom AC ecommerce integrations.

---

Next: [Step 4 — Create an abandoned cart](../04-abandoned-cart)
