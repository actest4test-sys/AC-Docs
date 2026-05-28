---
title: Ecommerce
layout: default
nav_order: 2
has_children: true
permalink: /ecommerce/
---

# Ecommerce

**Connection → Customer → Order → Abandoned Cart**

This category walks through the four core REST API calls needed to push ecommerce data from a custom integration into ActiveCampaign: creating a store connection, attaching a customer, creating an order, and creating an abandoned cart. Use it alongside the official [ActiveCampaign Developer Docs](https://developers.activecampaign.com/reference/overview).

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

1. **[Step 1 — Create a connection](./01-connection)** — represents the external ecommerce store.
2. **[Step 2 — Create a customer](./02-customer)** — links a contact in AC to a customer record in that store.
3. **[Step 3 — Create an order](./03-order)** — a completed purchase tied to the customer and connection.
4. **[Step 4 — Create an abandoned cart](./04-abandoned-cart)** — same endpoint as orders, but with cart-specific fields. Update later when (or if) the cart converts.

---

## Reference links

- [API overview](https://developers.activecampaign.com/reference/overview)
- [Authentication](https://developers.activecampaign.com/reference/authentication)
- [Rate limits](https://developers.activecampaign.com/reference/rate-limits)
- [DeepData — Connections](https://developers.activecampaign.com/reference/connections)
- [DeepData — Ecommerce Customers](https://developers.activecampaign.com/reference/customers)
- [DeepData — Ecommerce Orders](https://developers.activecampaign.com/reference/orders)
- [DeepData — Abandoned Carts](https://developers.activecampaign.com/reference/e-commerce-abandoned-carts)
- [Abandoned cart automation (AC help center)](https://help.activecampaign.com/hc/en-us/articles/360001046024-How-do-I-create-an-abandoned-cart-automation)
