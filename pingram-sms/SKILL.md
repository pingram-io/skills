---
name: pingram-sms
description: Send SMS notifications with Pingram. Use when the user wants to send text messages, configure SMS settings, or set up two-way SMS communication.
---

# Pingram SMS

Send SMS notifications globally with Pingram's A2P-enabled infrastructure.

## Sending SMS

```typescript
import { Pingram } from 'pingram';

const client = new Pingram({ apiKey: 'pingram_sk_...' });

await client.send({
  type: 'verification_code',
  to: { number: '+15005550006' }, // E.164 format required
  sms: {
    message: 'Your verification code is 123456. Valid for 10 minutes.'
  }
});
```

**Note:** The `id` field in `to` is optional. If your system tracks users by ID, include it. Otherwise, `number` alone is sufficient and will also serve as the user identifier.

**Debugging:** Check the API response for errors. For delivery issues, use **Dashboard > Logs** to see the full message lifecycle.

## Phone Number Format

Always use E.164 format for phone numbers:

- US: `+15005550006`
- UK: `+447911123456`
- International: `+` followed by country code and number

```typescript
// Correct
to: {
  number: '+15005550006';
}

// Incorrect - missing country code
to: {
  number: '5005550006';
}

// Incorrect - formatting characters not allowed
to: {
  number: '(500) 555-0006';
}
```

## Auto-Reply

Configure automatic replies when users respond to your SMS:

```typescript
await client.send({
  type: 'survey',
  to: { id: 'user_123', number: '+15005550006' },
  sms: {
    message: 'How was your experience? Reply 1-5.',
    autoReply: {
      message: 'Thanks for your feedback! We appreciate it.'
    }
  }
});
```

## Two-Way SMS

> **Note:** Inbound SMS requires a paid account with a dedicated phone number. Contact support to set up your dedicated number.

Receive SMS replies via webhooks:

1. Set up a dedicated phone number (paid accounts only)
2. Configure an inbound webhook in **Settings > Webhooks**
3. Users reply to your SMS
4. Pingram forwards the reply to your webhook

Webhook payload:

```json
{
  "eventType": "SMS_INBOUND",
  "from": "+15005550006",
  "to": "+18885551234",
  "text": "Yes, confirm my appointment",
  "receivedAt": "2024-01-15T10:30:00Z",
  "userId": "user@example.com",
  "lastTrackingId": "019abc12-3456-7890-abcd-ef1234567890"
}
```

## A2P Compliance

Pingram uses A2P (Application-to-Person) enabled numbers for reliable delivery:

- **A2P 10DLC:** US long codes registered for business messaging
- **Short codes:** High-throughput numbers for large volumes
- **Toll-free:** US/Canada toll-free numbers

### Automatic Opt-Out Handling

Pingram automatically handles opt-out keywords. When a user replies with STOP, STOPALL, UNSUBSCRIBE, CANCEL, END, or QUIT, they are automatically suppressed from future messages. No action required on your part.

### Compliance Best Practices

1. **Get consent:** Only send to users who opted in
2. **Include opt-out:** "Reply STOP to unsubscribe"
3. **Identify yourself:** Include your company name
4. **Respect quiet hours:** Consider recipient timezone
5. **No spam:** Send relevant, expected messages

## SMS with Other Channels

Send to multiple channels simultaneously:

```typescript
await client.send({
  type: 'urgent_alert',
  to: {
    id: 'user_123',
    email: 'user@example.com',
    number: '+15005550006'
  },
  email: {
    subject: 'Urgent Alert',
    html: '<p>Detailed information here...</p>'
  },
  sms: {
    message: 'Urgent: Check your email for important details.'
  }
});
```

## Force SMS Only

```typescript
await client.send({
  type: 'otp',
  to: { id: 'user_123', number: '+15005550006' },
  sms: { message: 'Your code is 123456' }
});
```

## Delivery Status

Track SMS delivery via webhooks. See the [pingram-webhooks](./pingram-webhooks/SKILL.md) skill for full webhook setup.

```json
{
  "eventType": "SMS_DELIVERED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "verification_code",
  "channel": "SMS",
  "userId": "user@example.com"
}
```

Webhook event types: `SMS_DELIVERED`, `SMS_FAILED`, `SMS_UNSUBSCRIBE`

## Common Issues

**First steps for any issue:**

1. Check the API response for error messages
2. Go to **Dashboard > Logs** to see the full message lifecycle and status
3. Verify your API key is valid

**Invalid phone number:**

- Ensure E.164 format (`+` followed by country code and number)
- Remove spaces, dashes, and parentheses

**Message not delivered:**

- Check **Dashboard > Logs** for delivery status and error details
- Verify the number is valid and can receive SMS
- Check the number isn't on a suppression list

**Carrier filtering:**

- Include opt-out instructions
- Avoid spam trigger words
- Ensure A2P registration is complete
