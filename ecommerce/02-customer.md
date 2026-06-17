---
title: "Step 2 — Customer"
layout: default
parent: Ecommerce
nav_order: 2
permalink: /ecommerce/02-customer/
---

# Step 2 — Create an ecommerce customer

A customer record links an AC contact (by email) to a connection. Orders attach to customers, and the customer aggregate (lifetime revenue, order count, etc.) is calculated from this object. Full reference: [Create a customer](https://developers.activecampaign.com/reference/create-customer).

```
POST https://{youraccountname}.api-us1.com/api/3/ecomCustomers
```

## Request body

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

## Key fields

- `connectionid` — the `id` returned from [Step 1](../01-connection).
- `externalid` — the customer's ID in your store. Must be unique per connection.
- `email` — required. AC will create a contact with this email if one doesn't already exist.
- `acceptsMarketing` — `"1"` or `"0"`. Reflects marketing consent in the source system.

> **⚠ If the customer already exists**
>
> Creating a customer with the same `connectionid` + `externalid` combo will return a `422` error. `GET` the customer first (filter on `externalid` and `connectionid`) before attempting to create.

---

Next: [Step 3 — Create an order](../03-order)
