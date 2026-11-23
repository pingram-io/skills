---
name: pingram-group-texts
description: Send one text to several numbers with Pingram. Use when the user wants a group text, a text to 2–8 numbers in the US or Canada, or group text delivery and replies.
---

# Pingram group texts

Send one text to 2–8 numbers in the US or Canada. Phones keep a later text to the same numbers in the same chat. One-to-one texts stay on `sms.send` (`POST /sms`).

## Sending a group text

`sms.sendGroup` (`POST /sms/group`).

```typescript
import { Pingram } from 'pingram';

const client = new Pingram({ apiKey: 'pingram_sk_...' });

await client.sms.sendGroup({
  type: 'onboarding',
  to: ['+14165550100', '+16045550100'],
  message: 'Welcome to the project chat.',
  subject: 'Project chat'
});
```

`type` is a notification type you can use later for filtering, analytics, and more. For example, `onboarding` or `threshold_alert`.

`to` is 2 to 8 mobile numbers in the US or Canada, in E.164. Each number is billed at the multimedia message rate, including a text with no attachment. The cost counts toward the texting budget. Include each number only once.

`message` is required when `mediaUrls` is omitted. `subject` is optional and is shown with the text. `mediaUrls` are public HTTPS URLs of pictures or files.

`from` is the number the group text is sent from. When omitted, a number on this account is used. The account needs a number of its own. A group text cannot be scheduled. A follow-up is another `sms.sendGroup` to the same numbers. Each send has its own tracking id. The response is the same tracking-id shape as `POST /sms`.

## Delivery and replies

Each number that receives the group text sends `GROUP_TEXT_DELIVERED`. The first of those marks the group text delivered. `recipient` is the number that was delivered. `participants` is the numbers the text was sent to, in that order. The number on your account is `from`, and is not included there. `type` is the notification type from the send.

`GROUP_TEXT_FAILED` means the send itself was rejected. One number failing does not fail the group.

A message someone types in the group is `GROUP_TEXT_INBOUND` for each account that owns a number in the group. It is not tied to an earlier send. STOP and START in that group are still `SMS_UNSUBSCRIBE` and `SMS_SUBSCRIBE`.

Webhook setup is in [pingram-webhooks](../pingram-webhooks/SKILL.md). Replies are in [pingram-inbound](../pingram-inbound/SKILL.md).

## Logs

Group texts are on **Dashboard > Logs**, under the filter card as Group Text Logs, once this account has at least one. That opens its own page. They are not on the one-to-one delivery status. List them with `logs.listGroupTexts` (`GET /logs/group-texts`). Load rows with `logs.getGroupTextLog` (`GET /logs/group-texts/{trackingIds}`).
