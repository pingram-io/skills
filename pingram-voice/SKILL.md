---
name: pingram-voice
description: Send voice call notifications with Pingram. Use when the user wants to make automated phone calls, text-to-speech notifications, or voice alerts.
---

# Pingram Voice Calls

Send automated voice calls with text-to-speech for urgent notifications, verification codes, and alerts.

## Sending Voice Calls

```typescript
import { Pingram } from 'pingram';

const client = new Pingram({ apiKey: 'pingram_sk_...' });

await client.send({
  type: 'urgent_alert',
  to: { number: '+15005550006' }, // E.164 format required
  call: {
    message:
      'This is an urgent alert from Acme Company. Please check your email for important information.'
  }
});
```

**Note:** The `id` field in `to` is optional. If your system tracks users by ID, include it. Otherwise, `number` alone is sufficient and will also serve as the user identifier.

**Debugging:** Check the API response for errors. For delivery issues, use **Dashboard > Logs** to see the full call lifecycle.

## Phone Number Format

Use E.164 format for all phone numbers:

- US: `+15005550006`
- UK: `+447911123456`
- International: `+` followed by country code and number

## Text-to-Speech Tips

Optimize your messages for spoken delivery:

### Pace and Clarity

```typescript
// Good: Spaced for clarity
call: {
  message: 'Your code is 1, 2, 3, 4, 5, 6.';
}

// Less clear: Numbers run together
call: {
  message: 'Your code is 123456.';
}
```

### Spelling Out Information

```typescript
// Good: Spell out important details
call: {
  message: 'Your confirmation number is A as in Alpha, B as in Bravo, 1, 2, 3.';
}

// Less clear
call: {
  message: 'Your confirmation number is AB123.';
}
```

### Repetition for Important Information

```typescript
call: {
  message: 'Your verification code is 1, 2, 3, 4, 5, 6. I repeat, your code is 1, 2, 3, 4, 5, 6.';
}
```

### Natural Pauses

Use commas and periods to create natural pauses:

```typescript
call: {
  message: 'Hello. This is an important message from Acme Company. Your account requires attention. Please log in to your account, or call us at 1, 800, 555, 1234.';
}
```

## Combining with Other Channels

Send voice calls alongside other notifications:

```typescript
await client.send({
  type: 'urgent_alert',
  to: {
    id: 'user_123',
    email: 'user@example.com',
    number: '+15005550006'
  },
  email: {
    subject: 'Urgent: Action Required',
    html: '<p>Detailed information here...</p>'
  },
  sms: {
    message: 'Urgent alert - check your email or we will call you shortly.'
  },
  call: {
    message:
      'This is an urgent alert. Please check your email for important information.'
  }
});
```

## Force Voice Only

```typescript
await client.send({
  type: 'verification',
  to: { id: 'user_123', number: '+15005550006' },
  call: { message: 'Your code is 1, 2, 3, 4, 5, 6.' }
});
```

## Delivery Status

Track call failures via webhooks. See the [pingram-webhooks](./pingram-webhooks/SKILL.md) skill for full webhook setup.

```json
{
  "eventType": "CALL_FAILED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "urgent_alert",
  "channel": "CALL",
  "userId": "user@example.com",
  "failureCode": "NO_ANSWER"
}
```

Webhook event types: `CALL_FAILED`, `CALL_UNSUBSCRIBE`

> **Note:** Successful call completions are logged in **Dashboard > Logs** but do not trigger webhooks.

## Best Practices

1. **Keep messages concise:** 30-60 seconds is ideal
2. **Repeat important information:** Codes, numbers, dates
3. **Use natural speech patterns:** Include pauses with punctuation
4. **Identify yourself:** State your company name at the start
5. **Provide alternatives:** "Press any key to repeat" or callback numbers
6. **Respect calling hours:** Consider recipient timezone
7. **Test your messages:** Call yourself to verify pronunciation

## Common Issues

**First steps for any issue:**

1. Check the API response for error messages
2. Go to **Dashboard > Logs** to see the full call lifecycle and status
3. Verify your API key is valid and has voice permissions

**Call not connecting:**

- Check **Dashboard > Logs** for call status and error details
- Verify E.164 phone number format
- Check the number is valid and accepts calls
- User may have call blocking enabled

**Poor pronunciation:**

- Add spaces between digits: `1, 2, 3` not `123`
- Spell out abbreviations
- Use punctuation for pauses

**Message too fast:**

- Add commas for natural pauses
- Repeat important information
