---
name: pingram-inbound
description: Receive inbound emails and SMS with Pingram. Use when the user wants to receive messages, set up reply handling, configure inbound webhooks, or enable two-way communication.
---

# Pingram Inbound Messages

Receive emails and SMS messages from your users and handle them in your application via webhooks.

## Overview

Pingram supports inbound messaging for:

- **Email:** Receive emails at your custom domain or `yourcompany@mail.pingram.io`
- **SMS:** Receive SMS replies to outbound messages

## Email Inbound

### Default Inbox

Every Pingram account includes an inbox:

```
yourcompany@mail.pingram.io
```

Emails sent to this address are forwarded to your webhook.

### Custom Domain Inbound

To receive emails at your own domain (e.g., `support@yourcompany.com`):

1. **Verify your domain** in Settings > Domains
2. **Add MX record:** Copy the MX record shown in the dashboard and add it to your DNS provider

3. **Configure webhook** in Settings > Webhooks

### Inbound Email Webhook

Configure your webhook URL to receive inbound emails:

```json
{
  "eventType": "EMAIL_INBOUND",
  "from": "customer@example.com",
  "fromName": "John Doe",
  "to": "support@yourcompany.com",
  "cc": ["manager@yourcompany.com"],
  "replyTo": "customer@example.com",
  "subject": "Need help with my order",
  "bodyText": "Plain text version of the email body",
  "bodyHtml": "<p>HTML version of the email body</p>",
  "attachments": [
    {
      "filename": "screenshot.png",
      "contentType": "image/png",
      "size": 12345,
      "content": "base64-encoded-content"
    }
  ],
  "messageId": "<unique-id@example.com>",
  "inReplyTo": "<original-message-id>",
  "references": "<thread-references>",
  "receivedAt": "2024-01-15T10:30:00Z",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "userId": "customer@example.com",
  "type": "support_inquiry"
}
```

### Reply Detection

Pingram automatically detects replies to your outbound emails:

- `trackingId` - links the reply to the original outbound notification
- `type` - the notification type of the original message
- `userId` - the recipient of the original notification
- `inReplyTo` and `references` - standard email threading headers

Use this for:

- Support ticket systems
- Conversation threading
- Reply-based workflows

## SMS Inbound

### Receiving SMS Replies

When users reply to SMS notifications, Pingram forwards the message to your webhook. STOP and START are not `SMS_INBOUND`; they arrive as `SMS_UNSUBSCRIBE` and `SMS_SUBSCRIBE`. HELP is a normal inbound message.

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

### Two-Way SMS Conversations

```typescript
// Handle inbound SMS
app.post('/webhooks/pingram', async (req, res) => {
  const { eventType, from, text, userId, lastTrackingId } = req.body;

  if (eventType === 'SMS_INBOUND') {
    // Process the reply
    if (text.toLowerCase() === 'yes') {
      // Confirm action and respond
      await client.send({
        type: 'confirmation',
        to: { number: from },
        sms: { message: 'Great! Your appointment is confirmed.' }
      });
    } else if (text.toLowerCase() === 'no') {
      await client.send({
        type: 'cancellation',
        to: { number: from },
        sms: { message: 'Your appointment has been cancelled.' }
      });
    }
  }

  res.status(200).send('OK');
});
```

### SMS Auto-Reply

Configure automatic replies when sending:

```typescript
await client.send({
  type: 'survey',
  to: { id: 'user_123', number: '+15005550006' },
  sms: {
    message: 'How was your experience? Reply 1-5.',
    autoReply: {
      message: 'Thanks for your feedback!'
    }
  }
});
```

## Webhook Configuration

### Setting Up Webhooks

1. Go to **Settings > Webhooks** in the dashboard
2. Click **Add Webhook**
3. Enter your endpoint URL
4. Select events to receive:
   - `EMAIL_INBOUND`
   - `SMS_INBOUND`
5. Save and test
