---
name: pingram-webhooks
description: Configure webhooks for Pingram events. Use when the user wants to receive delivery notifications, track email opens and clicks, handle bounces, or integrate with external systems.
---

# Pingram Webhooks

Receive real-time notifications about your messages via webhooks. Track delivery, opens, clicks, bounces, and more.

## Setting Up Webhooks

1. Go to **Webhook** in the Pingram dashboard
2. Click **Add endpoint**
3. Enter your endpoint URL (must be HTTPS)
4. Select the events you want that endpoint to receive
5. Save. Repeat to add more endpoints. Each endpoint has its own signing secret.

## API

```
GET    /webhooks
POST   /webhooks
PUT    /webhooks/{endpointId}
DELETE /webhooks/{endpointId}
```

SDK methods: `webhooks.listWebhooks`, `webhooks.createWebhook`, `webhooks.updateWebhook`, `webhooks.deleteWebhook`.

## Event Types

### Email Events

| Event Type          | Description                           |
| ------------------- | ------------------------------------- |
| `EMAIL_DELIVERED`   | Email delivered to recipient's server |
| `EMAIL_FAILED`      | Email delivery failed                 |
| `EMAIL_OPEN`        | Recipient opened the email            |
| `EMAIL_CLICK`       | Recipient clicked a link              |
| `EMAIL_UNSUBSCRIBE` | Recipient unsubscribed                |
| `EMAIL_INBOUND`     | Received an inbound email             |

### SMS Events

| Event Type        | Description                                               |
| ----------------- | --------------------------------------------------------- |
| `SMS_DELIVERED`   | SMS delivered to recipient                                |
| `SMS_FAILED`      | SMS delivery failed                                       |
| `SMS_UNSUBSCRIBE` | Recipient replied STOP (further SMS to that number stops) |
| `SMS_SUBSCRIBE`   | Recipient replied START (SMS to that number resumes)      |
| `SMS_INBOUND`     | Received an inbound SMS, including HELP                   |

### Voice Events

| Event Type         | Description            |
| ------------------ | ---------------------- |
| `CALL_FAILED`      | Voice call failed      |
| `CALL_UNSUBSCRIBE` | Recipient unsubscribed |

> **Note:** Inbound events (`EMAIL_INBOUND`, `SMS_INBOUND`) have a different payload structure. See the pingram-inbound skill for details.

## Webhook Payload Structure

All webhook payloads use a flat structure with the following fields:

```json
{
  "eventType": "EMAIL_DELIVERED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "welcome_email",
  "channel": "EMAIL",
  "userId": "user@example.com"
}
```

| Field             | Type   | Description                                                      |
| ----------------- | ------ | ---------------------------------------------------------------- |
| `eventType`       | string | The event type (see Event Types above)                           |
| `trackingId`      | string | Unique ID for the notification (optional)                        |
| `notificationId`  | string | The notification type                                            |
| `channel`         | string | `EMAIL`, `SMS`, `CALL`                                           |
| `userId`          | string | The recipient's user ID                                          |
| `failureCode`     | string | Error code (only for `*_FAILED` events)                          |
| `clickedLink`     | string | URL clicked (only for `EMAIL_CLICK`)                             |
| `clickedLinkTags` | object | Link tags (only for `EMAIL_CLICK`)                               |

> **Note:** Inbound events (`EMAIL_INBOUND`, `SMS_INBOUND`) have a different payload structure with fields like `from`, `to`, `text`/`bodyText`, `receivedAt`, etc. See the [pingram-inbound](./pingram-inbound/SKILL.md) skill for detailed payload examples.

## Event Payloads

### EMAIL_DELIVERED

```json
{
  "eventType": "EMAIL_DELIVERED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "welcome",
  "channel": "EMAIL",
  "userId": "user@example.com"
}
```

### EMAIL_FAILED

```json
{
  "eventType": "EMAIL_FAILED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "welcome",
  "channel": "EMAIL",
  "userId": "user@example.com",
  "failureCode": "EMAIL_BOUNCE"
}
```

### EMAIL_OPEN

```json
{
  "eventType": "EMAIL_OPEN",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "welcome",
  "channel": "EMAIL",
  "userId": "user@example.com"
}
```

### EMAIL_CLICK

```json
{
  "eventType": "EMAIL_CLICK",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "welcome",
  "channel": "EMAIL",
  "userId": "user@example.com",
  "clickedLink": "https://app.example.com/activate",
  "clickedLinkTags": {
    "campaign": ["onboarding"],
    "action": ["activate"]
  }
}
```

### SMS_DELIVERED

```json
{
  "eventType": "SMS_DELIVERED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "otp",
  "channel": "SMS",
  "userId": "user@example.com"
}
```

### SMS_FAILED

```json
{
  "eventType": "SMS_FAILED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "otp",
  "channel": "SMS",
  "userId": "user@example.com",
  "failureCode": "INVALID_NUMBER"
}
```

### CALL_FAILED

```json
{
  "eventType": "CALL_FAILED",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "notificationId": "alert",
  "channel": "CALL",
  "userId": "user@example.com",
  "failureCode": "NO_ANSWER"
}
```

### \*\_UNSUBSCRIBE

All unsubscribe events follow this format:

```json
{
  "eventType": "EMAIL_UNSUBSCRIBE",
  "notificationId": "marketing",
  "channel": "EMAIL",
  "userId": "user@example.com"
}
```

## Webhook Handler Example

```typescript
import express from 'express';

const app = express();
app.use(express.json());

app.post('/webhooks/pingram', (req, res) => {
  const {
    eventType,
    trackingId,
    notificationId,
    channel,
    userId,
    failureCode,
    clickedLink
  } = req.body;

  switch (eventType) {
    case 'EMAIL_DELIVERED':
      console.log(`Email delivered to ${userId}`);
      // Update delivery status in your database
      break;

    case 'EMAIL_FAILED':
      console.log(`Email failed: ${failureCode}`);
      // Handle bounce, mark email as invalid if hard bounce
      break;

    case 'EMAIL_OPEN':
      console.log(`Email opened by ${userId}`);
      // Track engagement metrics
      break;

    case 'EMAIL_CLICK':
      console.log(`Link clicked: ${clickedLink}`);
      // Track click-through rates
      break;

    case 'SMS_DELIVERED':
      console.log(`SMS delivered to ${userId}`);
      break;

    case 'SMS_FAILED':
      console.log(`SMS failed: ${failureCode}`);
      break;

    case 'EMAIL_UNSUBSCRIBE':
    case 'SMS_UNSUBSCRIBE':
      console.log(`${userId} unsubscribed from ${channel}`);
      // Update user preferences
      break;

    default:
      console.log(`Unhandled event: ${eventType}`);
  }

  // Always respond quickly
  res.status(200).send('OK');
});

app.listen(3000);
```

## Best Practices

### 1. Respond Quickly

```typescript
app.post('/webhooks/pingram', async (req, res) => {
  // Respond immediately
  res.status(200).send('OK');

  // Process asynchronously
  processWebhookAsync(req.body).catch(console.error);
});
```

### 2. Handle Duplicates

Use `trackingId` for idempotency:

```typescript
app.post('/webhooks/pingram', async (req, res) => {
  const { eventType, trackingId } = req.body;
  const eventKey = `${eventType}:${trackingId}`;

  // Check if already processed
  if (await redis.exists(eventKey)) {
    return res.status(200).send('Already processed');
  }

  // Mark as processing
  await redis.setex(eventKey, 86400, '1');

  // Process event...
  res.status(200).send('OK');
});
```

### 3. Use Queues for Heavy Processing

```typescript
import { Queue } from 'bullmq';

const webhookQueue = new Queue('webhooks');

app.post('/webhooks/pingram', async (req, res) => {
  // Add to queue for processing
  await webhookQueue.add('process', req.body);
  res.status(200).send('OK');
});
```

## Testing Webhooks

### Using the Dashboard

1. Go to Settings > Webhooks
2. Click "Test" next to your webhook
3. Select an event type
4. Send test payload

### Local Development

Use a tunneling service like ngrok:

```bash
ngrok http 3000
```

Then use the ngrok URL as your webhook endpoint.

## Common Issues

**Not receiving webhooks:**

- Verify URL is correct and HTTPS
- Check firewall allows incoming connections
- Ensure your server returns 2xx status

**Duplicate events:**

- Implement idempotency using trackingId
- Store processed event IDs temporarily

**Webhook delays:**

- Check your server response time
- Ensure you're responding with 200 quickly
