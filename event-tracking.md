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

Event Tracking uses a **different endpoint and different credentials** from the main REST API.

| | REST API (everything else on this site) | Event Tracking |
|---|---|---|
| Endpoint | `https://{youraccountname}.api-us1.com/api/3/...` | `https://trackcmp.net/event` |
| Auth header | `Api-Token: YOUR_API_KEY` | Form params: `actid` + `key` |
| Content type | `application/json` | `application/x-www-form-urlencoded` |

**Two distinct credentials — do not mix them up:**

- **Event Key (`key`)** — a dedicated tracking key found under **Settings → Tracking → Event Tracking**. This is what you pass to `trackcmp.net/event`. It is **not** your API key and cannot be used in its place.
- **Account ID (`actid`)** — your numeric AC account ID, also shown on the same settings page.
- **Api-Token** — your standard REST API key (used for everything on this site: registering event names via `POST /api/3/eventTrackingEvents`, syncing contacts, etc.). The Api-Token is **never** passed to `trackcmp.net/event`.

In short: use the **Event Key + actid** for the event-tracking endpoint; use the **Api-Token** for all REST API calls.

> **Enable Event Tracking first**
>
> In the same settings page, toggle Event Tracking on. Calls to `trackcmp.net/event` will silently fail (returning `0`) until the feature is enabled on the account.

For more on event-tracking behavior and the in-app UI, see the [ActiveCampaign Help Center article on event tracking](https://help.activecampaign.com/hc/en-us/articles/221870128-An-overview-of-Event-Tracking).

---

## Register an event name

Event names should be registered before they're used. You can do this in the UI (**Settings → Tracking → Event Tracking → Add Event**) or via the REST API:

```
POST https://{youraccountname}.api-us1.com/api/3/eventTrackingEvents
```

```json
{
  "eventTrackingEvent": {
    "name": "Played Video"
  }
}
```

The corresponding `GET /api/3/eventTrackingEvents` lists registered events.

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
| `key` | yes | The Event Key (not your API key — see "Before you start" above). |
| `event` | yes | The event name (must match a registered event — see above). |
| `eventdata` | no | A single free-form string. Surfaces on the contact timeline and can be matched in automation triggers. |
| `visit` | yes | A **JSON-encoded string** identifying the contact, typically `{"email":"jane@example.com"}`. |

### Example request

```
POST https://trackcmp.net/event
Content-Type: application/x-www-form-urlencoded
```

```
actid=12345
key=YOUR_EVENT_KEY
event=Played Video
eventdata=Onboarding intro · 2:14
visit={"email":"jane@example.com"}
```

### Response

The response is a plain text body:

- `1` — event was accepted
- `0` — event was rejected (feature disabled, unknown event name, or invalid `actid`/`key`)

> **Lo-fi response shape**
>
> `trackcmp.net/event` doesn't return JSON or a detailed error. If you get `0`, the most common causes are: Event Tracking not enabled, wrong key (Api-Token used instead of Event Key), or unregistered event name.

---

## Identifying the contact

The `visit` parameter is the only way the event gets attached to a contact. Most integrations use `{"email": "..."}`.

For real-time pipelines, sync the contact via [`POST /api/3/contact/sync`](../contacts-lists-tags/01-sync-contact/) **before** firing the event.

---

## Triggering automations from events

In the automation builder, choose the **"Event is recorded"** trigger:

- **Event name** — must match exactly (case-sensitive in most cases).
- **Event data condition** *(optional)* — match the `eventdata` string with operators (contains, equals, starts with).

This is the primary mechanism for behavior-driven retention automations: every named event becomes a potential automation entry point.

---

## Common pitfalls

- **Using `Api-Token` instead of the Event Key.** They are distinct credentials — the Event Key comes from Settings → Tracking → Event Tracking, not the API settings page.
- **Forgetting to enable Event Tracking** in account settings. Calls silently return `0`.
- **Sending `visit` as a raw object instead of a JSON-encoded string.** It must be stringified.
- **Unregistered event names being dropped silently** on newer accounts. Register before firing.
- **Treating the response as JSON.** It's a plain `0` or `1`.
- **Sending the event before the contact exists.** Sync the contact first.

---

## Reference links

- [Event tracking developer reference (AC docs)](https://developers.activecampaign.com/reference/event-tracking)
- [Contact Event Tracking API guide (AC docs)](https://developers.activecampaign.com/reference/contact-event-tracking-api-guide)
- [Track event endpoint (AC docs)](https://developers.activecampaign.com/reference/track-event)
- [Event Tracking overview (AC help)](https://help.activecampaign.com/hc/en-us/articles/221870128-An-overview-of-Event-Tracking)
- [Using Event Tracking to trigger automations (AC help)](https://help.activecampaign.com/hc/en-us/articles/224653688-Using-Event-Tracking-to-trigger-automations)
