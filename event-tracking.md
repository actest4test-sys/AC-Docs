---
title: Event Tracking
layout: default
nav_order: 5
permalink: /event-tracking/
---

# Event Tracking

Send arbitrary named events ("Played Video", "Viewed Pricing", "Started Trial", "Cancelled Subscription") into ActiveCampaign and tie them to a contact. Events show up on the contact record and can trigger automations.

Event Tracking is **separate from Site Tracking** (the JS pixel that records page views). Event Tracking is for explicit, named business events you send from your backend or app.

---

## Before you start

Event Tracking uses a **different endpoint and a different credential** from the main REST API.

| | REST API (everything else on this site) | Event Tracking |
|---|---|---|
| Endpoint | `https://{youraccountname}.api-us1.com/api/3/...` | `https://trackcmp.net/event` |
| Auth header | `Api-Token: YOUR_API_KEY` | Form params: `actid` + `key` |
| Content type | `application/json` | `application/x-www-form-urlencoded` |

Find the **Account ID (`actid`)** and **Event Key (`key`)** under **Settings → Tracking → Event Tracking** in the AC UI. The Event Key is distinct from your API key — don't confuse them.

> **⚠ Enable Event Tracking first**
>
> In the same settings page, toggle Event Tracking on. Calls to `trackcmp.net/event` will silently fail (returning `0`) until the feature is enabled on the account.

---

## Register an event name

Event names should be registered before they're used. You can do this in the UI (**Settings → Tracking → Event Tracking → Add Event**) or via the REST API:

```
POST /api/3/eventTrackingEvents
```

```json
{
  "eventTrackingEvent": {
    "name": "Played Video"
  }
}
```

The corresponding `GET /api/3/eventTrackingEvents` lists registered events.

> **⚠ Auto-registration behaviour has changed over time**
>
> Older accounts auto-registered new event names on first use. Newer accounts may silently drop unrecognised events. Register explicitly to be safe.

---

## Send an event

```
POST https://trackcmp.net/event
Content-Type: application/x-www-form-urlencoded
```

### Form parameters

| Param | Required | Description |
|---|---|---|
| `actid` | yes | Your Account ID from the Event Tracking settings page. |
| `key` | yes | The Event Key (not your API key). |
| `event` | yes | The event name (must match a registered event — see above). |
| `eventdata` | no | A single free-form string. Surfaces on the contact timeline and can be matched in automation triggers. |
| `visit` | yes | A **JSON-encoded string** identifying the contact, typically `{"email":"jane@example.com"}`. |

### Example (curl)

```bash
curl https://trackcmp.net/event \
  -d "actid=12345" \
  -d "key=YOUR_EVENT_KEY" \
  -d "event=Played Video" \
  -d "eventdata=Onboarding intro · 2:14" \
  --data-urlencode 'visit={"email":"jane@example.com"}'
```

### Example (JavaScript, server-side)

```js
const body = new URLSearchParams({
  actid: process.env.AC_ACTID,
  key:   process.env.AC_EVENT_KEY,
  event: "Played Video",
  eventdata: "Onboarding intro · 2:14",
  visit: JSON.stringify({ email: "jane@example.com" }),
});

await fetch("https://trackcmp.net/event", {
  method: "POST",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body,
});
```

### Response

The response is a plain text body:

- `1` — event was accepted
- `0` — event was rejected (feature disabled, unknown event name, or invalid `actid`/`key`)

> **⚠ Lo-fi response shape**
>
> `trackcmp.net/event` doesn't return JSON or a detailed error. If you get `0`, the most common causes are: Event Tracking not enabled, wrong key (API key vs event key), or unregistered event name.

---

## Identifying the contact

The `visit` parameter is the only way the event gets attached to a contact. Most integrations use `{"email": "..."}`. The contact must exist in AC first — if it doesn't, the event is still recorded but won't surface anywhere useful until a matching contact is created.

For real-time pipelines, sync the contact via [`POST /api/3/contact/sync`](../contacts-lists-tags/01-sync-contact/) **before** firing the event.

---

## Triggering automations from events

In the automation builder, choose the **"Event is recorded"** trigger:

- **Event name** — must match exactly (case-sensitive in most cases).
- **Event data condition** *(optional)* — match the `eventdata` string with operators (contains, equals, starts with).

This is the primary mechanism for behavior-driven retention automations: every named event becomes a potential automation entry point.

---

## Common pitfalls

- **Using `Api-Token` instead of the Event Key.** They are distinct credentials.
- **Forgetting to enable Event Tracking** in account settings. Calls silently return `0`.
- **Sending `visit` as a raw object instead of a JSON-encoded string.** It must be stringified.
- **Unregistered event names being dropped silently** on newer accounts. Register before firing.
- **Treating the response as JSON.** It's a plain `0` or `1`.
- **Sending the event before the contact exists.** Sync the contact first.

---

## Reference links

- [Event tracking developer reference (AC docs)](https://developers.activecampaign.com/reference/event-tracking)
- [Event tracking overview (AC help)](https://help.activecampaign.com/hc/en-us/articles/220117847-Event-Tracking)
- [Automation trigger: Event is recorded (AC help)](https://help.activecampaign.com/hc/en-us/articles/360024860914-Event-trigger)
