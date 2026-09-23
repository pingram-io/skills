---
name: pingram-users
description: Identify and manage users in Pingram. Use when syncing users, setting emails, phone numbers, timezones, or custom properties, segmenting broadcast audiences, listing or deleting users, or deciding what to pass in the `to` field of a send call.
---

# Pingram Users

Users are the recipients of notifications. Every user has a developer-chosen `id` (your own user ID); channel addresses (email, phone number) and custom properties hang off it. Users exist implicitly — you can send to `{ id, email }` without creating anything first — but syncing users with `identify` unlocks audience segmentation, preferences, and cleaner sends.

## Identify (Create or Update)

`POST /users/{userId}` upserts a user. Call it whenever a user signs up or their contact info changes:

```typescript
import { Pingram } from 'pingram';

const pingram = new Pingram({ apiKey: process.env.PINGRAM_API_KEY });

await pingram.user.identify('user-123', {
  email: 'dana@acme.io',
  number: '+15005550006', // E.164, for SMS/voice
  timezone: 'America/New_York',
  properties: {
    plan: 'pro',
    seats: 12,
    trial: false
  }
});
```

Identify merges: fields you omit are left untouched, and `properties` you send are merged key-by-key into the existing map. `lastSeenTime` updates automatically on every call.

Identifiable fields:

| Field           | Purpose                                                      |
| --------------- | ------------------------------------------------------------ |
| `email`      | Email notifications                    |
| `number`     | SMS and voice calls (E.164 format)     |
| `timezone`   | IANA timezone, used for scheduling     |
| `properties` | Custom key-value data for segmentation |

## Custom Properties

`properties` is a flat map of strings, numbers, and booleans — no nested objects or arrays. Limits:

- At most 25 keys per user
- Keys up to 64 characters
- String values up to 256 characters
- Whole map under 1 KB serialized

Properties power broadcast audience filters (see the `pingram-broadcasts` skill):

```typescript
audience: {
  filter: { 'properties.plan': { $eq: 'pro' } }
}
```

## Users and Sending

For transactional sends, identification is optional — `to` can carry the address inline:

```typescript
await client.send({
  type: 'welcome',
  to: { id: 'user-123', email: 'dana@acme.io' },
  email: { subject: 'Welcome!', html: '<p>Hi!</p>' }
});
```

If the user was identified earlier, `to: { id: 'user-123' }` is enough — Pingram resolves the stored email/number. Broadcasts with `audience.filter` only reach identified users.

## Read, List, Delete

```typescript
// Get one user (returns a bare { id } if never identified)
const user = await pingram.user.getUser('user-123');

// List users, paginated
const page = await pingram.users.listUsers(1000, '');
// page.users, page.hasMore; pass page.lastEvaluatedKey as the
// nextToken argument to fetch the next page

// Delete a user and all associated data (preferences)
await pingram.users.deleteUser('user-123');
```

REST equivalents: `GET /users/{userId}`, `GET /users?limit=&nextToken=`, `DELETE /users/{userId}`, all authenticated with `Authorization: Bearer $PINGRAM_API_KEY` against `https://api.pingram.io` (EU: `api.eu.pingram.io`, CA: `api.ca.pingram.io`).

## Notification Preferences

Users have per-notification-type, per-channel preferences, readable and writable over REST at `GET/POST /users/{userId}/preferences`. In most integrations you never touch them directly: unsubscribes from broadcast emails are recorded automatically when recipients click the `{{notificationapi:unsubscribe_url}}` link, and future broadcasts of that type skip them.

## Debugging

- **Broadcast filter matches nobody:** the users were never identified, or the property key/value differs (`plan: 'Pro'` vs `'pro'` — values are case-sensitive).
- **Properties rejected with 400:** check the caps above; nested objects and arrays are not allowed.
- **SMS/voice not delivering:** `number` must be E.164 (`+1…`), not a national format.
