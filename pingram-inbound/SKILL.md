---
name: pingram-inbound
description: Receive inbound emails and SMS with Pingram. Use when the user wants to receive messages, set up reply handling, configure inbound webhooks, or enable two-way communication.
---

# Pingram Inbound Messages

Receive emails and SMS messages from your users and handle them in your application via webhooks.

## Overview

Pingram supports inbound messaging for:

- **Email:** Receive every address on your domain and its subdomains, or try a Pingram address before you have a domain
- **SMS:** Receive SMS replies to outbound messages

## Email Inbound

### Your domain

Verify the domain in Settings > Domains and add the inbound MX record from the dashboard. Pingram then delivers every recipient on that domain and its subdomains to the `EMAIL_INBOUND` webhook. `support@yourcompany.com` and `scheduler@mail.yourcompany.com` both arrive when `yourcompany.com` is DKIM-verified. Adding a domain before DKIM succeeds does not deliver its mail. Do not create an address per mailbox.

### Trying inbound without a domain

Each account has one address for testing, shaped like `yourcompany@mail.pingram.io`. Mail to that exact address is delivered. These addresses are not for production traffic.

### Inbound Email Webhook

Subscribe an endpoint to `EMAIL_INBOUND` in Settings > Webhooks. Without that subscription, inbound mail is not sent to your application.

```json
{
  "eventType": "EMAIL_INBOUND",
  "from": "customer@example.com",
  "fromName": "John Doe",
  "matched": {
    "email": "support@yourcompany.com",
    "name": "Support"
  },
  "to": "client@example.com",
  "cc": ["manager@yourcompany.com"],
  "toRecipients": [
    { "email": "client@example.com" },
    { "email": "support@yourcompany.com", "name": "Support" }
  ],
  "ccRecipients": [{ "email": "manager@yourcompany.com", "name": "Manager" }],
  "replyTo": "customer@example.com",
  "subject": "Need help with my order",
  "bodyText": "Plain text version of the email body",
  "bodyHtml": "<p>HTML version of the email body</p>",
  "attachments": [
    {
      "filename": "screenshot.png",
      "contentType": "image/png",
      "size": 12345,
      "content": "base64-encoded-content",
      "contentId": "screenshot",
      "contentDisposition": "inline"
    }
  ],
  "messageId": "<unique-id@example.com>",
  "inReplyTo": "<original-message-id>",
  "references": "<thread-references>",
  "receivedAt": "2024-01-15T10:30:00Z",
  "sentAt": "2024-01-15T10:29:00Z",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890",
  "userId": "customer@example.com",
  "type": "support_inquiry"
}
```

`matched.email` is the address on your account that received the mail, including any address on your domain or its subdomains, and including when that address was only in Cc or Bcc. Compare it with `toRecipients`, `ccRecipients`, and `bccRecipients` to see which header contained it. `toRecipients` and `ccRecipients` are the original headers with display names. `to` and `cc` are deprecated and unchanged for existing webhooks. `to` is the first To-header address. `cc` is the Cc addresses without names. Use `toRecipients` and `ccRecipients`. `bccRecipients` is present only when the Bcc header is still on the message.

`receivedAt` is when the message was accepted. `sentAt` is the sender Date header and is absent when that header is missing or invalid.

`contentId` has the angle brackets removed, so it matches `cid:` references in `bodyHtml`. It is omitted when the attachment has no Content-ID. `contentDisposition` is `inline` or `attachment`.

The whole message, including headers, can be up to 40 MB. There is no separate per-attachment cap. Larger mail is bounced by the receiving service and is not stored or truncated.

One processed inbound email counts as 1 email. Attachment size does not add usage.

A failed inbound webhook can be resent for 30 days with `logs.retryInbound` (`POST /logs/retry/{trackingId}`). The resend keeps the original `trackingId` and does not count as another email.

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

Pingram forwards an inbound text to your webhook as `SMS_INBOUND`. `trackingId` identifies this inbound message. Inbound SMS to the free shared number only works when that person has already received a text from this number. Inbound SMS to a dedicated number works normally. STOP and START arrive as `SMS_UNSUBSCRIBE` and `SMS_SUBSCRIBE`. HELP is a normal inbound message.

```json
{
  "eventType": "SMS_INBOUND",
  "from": "+15005550006",
  "to": "+18885551234",
  "text": "Yes, confirm my appointment",
  "receivedAt": "2024-01-15T10:30:00Z",
  "userId": "user@example.com",
  "trackingId": "019abc12-3456-7890-abcd-ef1234567890"
}
```

### Two-Way SMS Conversations

```typescript
// Handle inbound SMS
app.post('/webhooks/pingram', async (req, res) => {
  const { eventType, from, text, userId, trackingId } = req.body;

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

## Webhook Configuration

### Setting Up Webhooks

Subscribe an http(s) endpoint to `EMAIL_INBOUND` and/or `SMS_INBOUND`.

Inbound SMS to the free shared number only works when that person has already received a text from this number. Inbound SMS to a dedicated number works normally.

**MCP:** `webhooks_createWebhook` with `webhook` and `events`. Save the returned `secret`. `webhooks_listWebhooks` shows existing endpoints. `webhooks_updateWebhook` replaces the full event list and keeps the secret.

**CLI:**

```bash
pingram webhooks create \
  --webhook https://example.com/webhooks/pingram \
  --events EMAIL_INBOUND SMS_INBOUND
```

**Dashboard:** Settings > Webhooks > Add endpoint, then select `EMAIL_INBOUND` and `SMS_INBOUND`.
