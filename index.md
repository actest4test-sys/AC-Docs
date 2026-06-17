---
title: Home
layout: default
nav_order: 1
---

# AC Docs

Integration guides for ActiveCampaign — written for solution engineers, partner developers, and anyone wiring an external system into AC.

Use the sidebar to navigate by category. Search is in the top-right.

---

## Authentication & base URL

### Authentication

All requests authenticate via an API key sent in the `Api-Token` header. You can find your key under **Settings → Developer** in the AC UI. Full details: [Authentication](https://developers.activecampaign.com/reference/authentication).

### Base URL

Every endpoint sits under your account-specific base URL:

```
https://{youraccountname}.api-us1.com/api/3/
```

Replace `{youraccountname}` with the subdomain shown in the AC URL bar. Reference: [Base URL](https://developers.activecampaign.com/reference/url).

### Standard headers

```
Api-Token: YOUR_API_KEY
Content-Type: application/json
Accept: application/json
```

These headers apply to all REST API calls across every section of this site. The [Event Tracking](./event-tracking/) section uses a different endpoint and different credentials — see that page for details.

---

## Categories

- **[Ecommerce](./ecommerce/)** — Connection, Customer, Order, Abandoned Cart
- **[Custom Objects](./custom-objects/)** — Schemas, Records, Querying
- **[Contacts, Lists & Tags](./contacts-lists-tags/)** — Sync, Subscribe, Tag, Custom Fields
- **[Event Tracking](./event-tracking/)** — Send named business events, trigger automations
- *(more coming soon)*
