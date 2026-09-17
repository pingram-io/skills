---
name: pingram-broadcasts
description: >-
  Draft and inspect Pingram email broadcasts for opted-in users. Use when
  creating or updating a draft newsletter, product update, or announcement,
  targeting an audience with user filters or known email lists, personalizing
  subject/HTML with Liquid mergetags, or reading campaign metrics. Never send
  or schedule a broadcast; a human must send from the Pingram dashboard.
  Pingram prohibits unsolicited messages.
---

# Pingram Broadcasts

Broadcasts are email campaigns to many recipients: newsletters, product updates, announcements. This skill **creates and edits drafts only**. After creating or updating a draft, give the user the `broadcastId`. They review audience and content in **Dashboard > Broadcast** and send it from there.

## Unsolicited email is prohibited

Pingram has strict policies against using broadcasts for unsolicited messages. Do not target purchased, scraped, rented, or otherwise non-consented lists. Only include people who opted in or already have a relationship with the sender. Always include `{{ pingram.unsubscribe }}`. Recipients who already unsubscribed from `type` are skipped automatically.

## Prerequisites

- A Pingram API key (`PINGRAM_API_KEY`), used as a Bearer token or via an SDK.
- A verified sending domain for full-speed sending. Broadcasts from unverified domains (including the shared default sender domain) are heavily throttled. Verify domains under **Dashboard > Settings > Domain Verification** (see the `pingram-email` skill).

## Create a Draft

`POST /broadcasts` creates a draft. Delivery does not start from this call.

```typescript
import { Pingram } from 'pingram';

const pingram = new Pingram({ apiKey: process.env.PINGRAM_API_KEY });

const broadcast = await pingram.broadcasts.create({
  name: 'August product update',
  type: 'product_updates',
  channel: 'email',
  audience: {
    filter: {
      $and: [
        { email: { $exists: true } },
        { 'properties.plan': { $eq: 'pro' } }
      ]
    }
  },
  fromName: 'Acme',
  fromAddress: 'updates@acme.com',
  replyToAddress: 'hello@acme.com',
  subject: 'What we shipped in August, {{ user.properties.name }}',
  html: '<p>Hi {{ user.properties.name | default: "there" }},</p><h1>August updates</h1><p><a href="{{ pingram.unsubscribe }}">Unsubscribe</a></p>'
});
// broadcast.broadcastId identifies the draft. A human sends it from the dashboard.
```

Field notes:

- `type` is the notification type key (e.g. `product_updates`); recipients can unsubscribe from it independently of other types. It is created automatically if it does not exist yet.
- `channel` is `'email'`.
- `subject` and `html` are Liquid templates rendered per recipient. See [Mergetags](#mergetags-per-recipient-liquid). Always include an unsubscribe link (`{{ pingram.unsubscribe }}`).

## Audience: Filter or Email List

`audience` takes exactly one of:

**`filter`** — a Mongo-style query evaluated against your synced users. Best for segments and large audiences:

```typescript
audience: {
  filter: { 'properties.plan': { $eq: 'pro' } }
}
```

Filter rules:

- Allowed fields: `email` and `properties.<key>` only.
- Allowed operators: `$and`, `$or`, `$exists`, `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`. Nothing else (`$regex`, `$not`, `$nor` are rejected).
- Caps: logical nesting depth <= 3, at most 20 clauses total.

Sync users first to unlock property filters (see the `pingram-users` skill for identification details):

```typescript
await pingram.user.identify('user-123', {
  email: 'dana@acme.io',
  properties: { plan: 'pro' }
});
```

**`emails`** — a raw list of addresses (max 10,000) that already opted in. Each address creates a user if one does not exist:

```typescript
audience: {
  emails: ['dana@acme.io', 'evan@acme.io'];
}
```

For audiences larger than 10,000, sync users via the Users API and use `filter`.

## Mergetags (per-recipient Liquid)

`subject` and `html` are rendered with [Liquid](https://liquidjs.com/) once per recipient. The context is the same user object you send to `identify` — custom fields stay under `user.properties`. They are **not** flattened onto `user` (`{{ user.name }}` is empty; use `{{ user.properties.name }}`).

```html
<p>Hi {{ user.properties.name | default: "there" }},</p>
<p>Your {{ user.properties.plan }} plan is active.</p>
{% if user.properties.trial %}
<p>Your trial is still running.</p>
{% endif %}
<p><a href="{{ pingram.unsubscribe }}">Unsubscribe</a></p>
```

| Tag | Source |
| --- | --- |
| `{{ user.id }}` | Your user id. For `audience.emails`, this is the lowercased email. |
| `{{ user.email }}` | Email address |
| `{{ user.number }}` | E.164 phone, if identified |
| `{{ user.timezone }}` | IANA timezone, if identified |
| `{{ user.properties.<key> }}` | Custom properties from `user.identify` (flat strings / numbers / booleans) |
| `{{ pingram.unsubscribe }}` | Per-recipient unsubscribe URL for this broadcast's `type` |
| `{{notificationapi:unsubscribe_url}}` | Legacy alias for the same URL. Still works; prefer `pingram.unsubscribe`. |

Notes:

- Missing properties render as empty — they do not fail the send. Use `| default:` when you need a fallback.
- Standard Liquid (`if` / `unless` / `else`, filters like `default`, `upcase`, `downcase`) is supported. Keep logic small; this is a template, not a program.
- HTML tags inside `{{ … }}` are stripped before render. Write `<strong>{{ user.properties.name }}</strong>`, not `{{ <strong>user.properties.name</strong> }}`.
- `audience.emails` only guarantees `user.id` and `user.email`. Property mergetags need those users identified first (see the `pingram-users` skill).
- Always include `{{ pingram.unsubscribe }}`. Recipients who click it are skipped on later broadcasts of the same `type`.

## Track Progress

Poll metrics after a human has started the send:

```typescript
const metrics = await pingram.broadcasts.getMetrics(broadcast.broadcastId);
// { status, total, sent, delivered, opened, clicked,
//   bounced, complained, unsubscribed, skipped, failed, ... }
```

`status` moves `draft -> sending -> sent` (or `scheduled`, `paused`, `canceled`). If the broadcast pauses itself (e.g. bounce/complaint guard), `pausedReason` explains why.

List per-recipient results, optionally filtered by status (`pending`, `sent`, `delivered`, `opened`, `clicked`, `bounced`, `complained`, `unsubscribed`, `skipped`, `failed`, `problems`):

```typescript
const recipients = await pingram.broadcasts.listRecipients(
  broadcast.broadcastId,
  'bounced'
);
```

## Manage Drafts

```typescript
await pingram.broadcasts.update(broadcastId, { subject: 'New subject' }); // drafts only
await pingram.broadcasts.list(); // newest first, paginated
await pingram.broadcasts.cancel(broadcastId); // stop a scheduled/sending/paused broadcast
await pingram.broadcasts.pause(broadcastId); // pause an active send
```

Only drafts can be edited. Scheduled or actively sending broadcasts are locked; cancel first (or have the user cancel in the dashboard) before editing.

## REST Endpoints

If not using an SDK:

| Action       | Endpoint                                   |
| ------------ | ------------------------------------------ |
| Create draft | `POST /broadcasts`                         |
| Update draft | `PATCH /broadcasts/{broadcastId}`          |
| List         | `GET /broadcasts`                          |
| Get          | `GET /broadcasts/{broadcastId}`            |
| Metrics      | `GET /broadcasts/{broadcastId}/metrics`    |
| Recipients   | `GET /broadcasts/{broadcastId}/recipients` |
| Pause        | `POST /broadcasts/{broadcastId}/pause`     |
| Cancel       | `POST /broadcasts/{broadcastId}/cancel`    |

Base URL: `https://api.pingram.io` (EU: `https://api.eu.pingram.io`, CA: `https://api.ca.pingram.io`). Authenticate with `Authorization: Bearer $PINGRAM_API_KEY`.

## Debugging

- **Broadcast paused unexpectedly:** check `pausedReason` in metrics. High bounce/complaint rates trigger an automatic guard.
- **Slow delivery:** the sending domain is likely unverified — verify it for full-speed sending.
- **Empty audience:** filter audiences require synced users; call `user.identify` first, or use `audience.emails` for opted-in addresses.
- **Mergetag rendered empty:** the path is wrong (`{{ user.name }}` instead of `{{ user.properties.name }}`), the user was never identified, or the property key/value differs (values are case-sensitive).
- Full message lifecycle per recipient is visible under **Dashboard > Logs**.
