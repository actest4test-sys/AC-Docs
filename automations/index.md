---
title: Automations
layout: default
nav_order: 6
has_children: true
permalink: /automations/
---

# Automations — Tracking Contact Journeys

**Find → Log → Map → Reconstruct**

Use the REST API to inspect which automations a contact has entered and reconstruct the steps they took. Three endpoints combine to give you a complete picture of any contact's path through any automation.

---

## What you'll learn

The AC automation API doesn't expose a single "show me everything this contact did" endpoint. Instead, it gives you three building blocks:

1. The list of automations a contact has entered (`contactAutomations[]` on the contact record)
2. An ordered log of the steps they passed through (`automationLogs`)
3. The block definitions for the automation itself (`automations/{id}/blocks`)

Joining these three gives you a full, timestamped journey — including branches taken, tags applied, and the block where the contact currently sits.

---

## Why it's useful

- **Per-contact activity timelines** — support and CS teams can see exactly what an automation did to a contact and when.
- **Branch-level funnel analysis** — which IF/ELSE branches are contacts actually taking, and at what rate?
- **Compliance and audit trails** — determine when a contact was tagged, scored, or moved to a list, and which automation caused it.
- **Stalled-contact detection** — identify contacts stuck on a wait block beyond an expected threshold so you can intervene.

---

## The three endpoints at a glance

| Purpose | Endpoint | What it returns |
|---|---|---|
| Find which automations a contact has entered | `GET /api/3/contacts/{id}` | `contactAutomations[]` — one entry per automation-enrolment |
| Get the step-by-step log | `GET /api/3/contactAutomations/{id}/automationLogs` | Ordered log of blocks the contact passed through |
| Get the automation's block definitions | `GET /api/3/automations/{id}/blocks` | Full list of steps (type, params, parent/child relationships) |

> **Note on endpoint IDs**
>
> The `{id}` in the log URL is the **contactAutomation id** (from the `contactAutomations[]` array), not the automation id. These are different numbers. Step 1 covers this in detail.

---

## The catch — read this first

The automation log reliably captures `send`, `wait`, and `goal` blocks, but **does not log**:

- `addtag`, `removetag`
- `scorecontact`
- `goto`
- `if` and `else` evaluations

For other block types (deal actions, `sub`/`unsub`, `enter`/`exit`, etc.) logging behaviour has not been fully verified — check per-type behaviour for your own automations.

Un-logged blocks do execute and have real side effects on the contact (tags added, scores changed, list memberships updated). The log just doesn't record them. Full path reconstruction therefore requires joining the log with the block definitions and reading the contact's side-effect history. [Step 4](./step-4-reconstruct-path) explains the full recipe.

---

## Walkthrough

1. **[Step 1 — Find the contact](./step-1-find-contact)** — retrieve `contactAutomations[]` and identify the enrolment IDs.
2. **[Step 2 — Read the automation log](./step-2-read-the-log)** — fetch the ordered log for a given enrolment.
3. **[Step 3 — Map log entries to block definitions](./step-3-map-to-blocks)** — fetch block definitions and join them to log entries.
4. **[Step 4 — Reconstruct the full path](./step-4-reconstruct-path)** — combine log, block graph, and contact side effects to build the complete journey.
5. **[Reference](./reference)** — endpoint cheat-sheet, field glossary, and known limitations.

---

## Reference links

- [Contacts endpoint (developer docs)](https://developers.activecampaign.com/reference/get-contact)
- [Contact automations (developer docs)](https://developers.activecampaign.com/reference/list-all-contact-automations)
- [Automation blocks (developer docs)](https://developers.activecampaign.com/reference/list-automations)
- [API authentication](https://developers.activecampaign.com/reference/authentication)
