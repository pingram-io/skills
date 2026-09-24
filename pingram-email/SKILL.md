---
name: pingram-email
description: Configure email delivery with Pingram including domains, senders, inbound email, outbound email, and SMTP relay. Use when setting up email sending, custom domains, SPF/DKIM, receiving emails, or SMTP integration.
---

# Pingram Email

Pingram provides complete email infrastructure: outbound sending, inbound receiving, custom domains, and SMTP relay.

## Outbound Email

### Sending Emails

```typescript
await client.send({
  type: 'welcome',
  to: { email: 'user@example.com' },
  email: {
    subject: 'Welcome to Our App',
    html: '<h1>Welcome!</h1><p>Thanks for signing up.</p>',
    senderName: 'Acme Team',
    senderEmail: 'hello@acme.com',
    previewText: 'Thanks for signing up'
  }
});
```

**Note:** The `id` field in `to` is optional. If your system tracks users by ID, include it. Otherwise, `email` alone is sufficient and will also serve as the user identifier.

**Debugging:** Check the API response for errors. For delivery issues, use **Dashboard > Logs** to see the full message lifecycle.

### Email Options

```typescript
await client.send({
  type: 'invoice',
  to: { email: 'user@example.com' },
  email: {
    subject: 'Your Invoice',
    html: '<p>Please find your invoice attached.</p>',
    senderName: 'Billing Team',
    senderEmail: 'billing@yourcompany.com'
  },
  options: {
    email: {
      replyToAddresses: ['support@yourcompany.com'],
      ccAddresses: ['manager@yourcompany.com'],
      bccAddresses: ['archive@yourcompany.com'],
      attachments: [
        {
          filename: 'invoice.pdf',
          url: 'https://storage.example.com/invoice.pdf'
        },
        {
          filename: 'terms.txt',
          content: 'base64content...',
          contentType: 'text/plain'
        }
      ]
    }
  }
});
```

### Attachment Options

**URL-based attachments:**

```typescript
attachments: [
  { filename: 'report.pdf', url: 'https://example.com/report.pdf' }
];
```

**Content-based attachments:**

```typescript
attachments: [
  {
    filename: 'data.csv',
    content: 'base64encodedcontent',
    contentType: 'text/csv'
  }
];
```

### Calendar invites

Send a calendar invite on the existing `send` call. Do not build a calendar object in Pingram. The caller supplies the iCalendar text.

Add one inline attachment on `options.email.attachments`:

- `filename`: `invite.ics`
- `content`: the iCalendar text, raw base64, no `data:` prefix
- `contentType`: `text/calendar; method=REQUEST`

`method` must match the `METHOD` line in the calendar (`REQUEST`, `CANCEL`, `REPLY`, or `PUBLISH`). That parameter is what makes Gmail and Outlook treat the file as an invite. A `.ics` filename with no `contentType` is only `text/calendar`. URL attachments cannot set `contentType`, so they cannot carry a method.

The calendar text needs `METHOD`, a stable `UID`, `DTSTART`, `DTEND`, `ORGANIZER`, and `ATTENDEE`, with CRLF line breaks. Reuse the `UID` to update the same event. Cancel with `METHOD:CANCEL` and `contentType` `text/calendar; method=CANCEL`.

### Size limits

- **Inline (`content`):** ~4 MB raw per attachment (~6 MB total request payload after base64 and JSON overhead). Returns 413 when exceeded. Works via API and SMTP.
- **URL (`url`):** Up to 20 MB per attachment. API only — not available via SMTP.
- For files over ~4 MB, use a URL attachment via the API.

## Custom Domains

### Why Use a Custom Domain?

- **Better deliverability:** Emails from your domain are less likely to be marked as spam
- **Brand consistency:** Send from `notifications@yourcompany.com` instead of a shared domain
- **Inbound email:** Receive replies at your domain

### Setting Up a Custom Domain

1. Go to **Settings > Domains** in the Pingram dashboard
2. Add your domain (e.g., `yourcompany.com`)
3. Copy the DNS records shown in the dashboard and add them to your DNS provider

### Adding Email Senders

After domain verification, add sender addresses:

1. Go to **Settings > Senders**
2. Click **Add Sender**
3. Enter the email address (e.g., `notifications@yourcompany.com`)
4. Set as default if desired

## Inbound Email

Pingram can receive emails and forward them to your application via webhooks.

### Default Inbox

Every account gets an inbox at `yourcompany@mail.pingram.io`. Configure a webhook to receive inbound emails.

### Custom Domain Inbound

To receive emails at your custom domain:

1. Verify your domain (see above)
2. Go to **Settings > Domains** and enable inbound for your domain
3. Add the MX record shown in the dashboard to your DNS provider
4. Configure your inbound webhook in **Settings > Webhooks**

### Inbound Webhook Payload

```json
{
  "eventType": "EMAIL_INBOUND",
  "from": "customer@example.com",
  "fromName": "John Doe",
  "to": "support@yourcompany.com",
  "cc": ["manager@yourcompany.com"],
  "replyTo": "customer@example.com",
  "subject": "Help needed",
  "bodyText": "Plain text body",
  "bodyHtml": "<p>HTML body</p>",
  "attachments": [
    {
      "filename": "document.pdf",
      "contentType": "application/pdf",
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

## SMTP Relay

Use Pingram as your SMTP server for sending emails from any application.

### SMTP Credentials

Get your SMTP credentials from **Settings > SMTP**:

| Setting  | Value                       |
| -------- | --------------------------- |
| Host     | `smtp.pingram.io`           |
| Port     | `587` (TLS) or `465` (SSL)  |
| Username | `type` (e.g. `auth_emails`) |
| Password | Your API key                |

## Common Issues

**First steps for any issue:**

1. Check the API response for error messages
2. Go to **Dashboard > Logs** to see the full message lifecycle and status
3. Verify your API key is valid

**Email not delivered:**

- Check **Dashboard > Logs** for delivery status and bounce details
- Verify the recipient address is valid
- Check if the domain is verified in **Settings > Domains**

**Email going to spam:**

- Ensure SPF, DKIM, and DMARC are configured (see **Settings > Domains**)
- Avoid spam trigger words in subject and body
- Include an unsubscribe link for marketing emails

**Bounced emails:**

- Hard bounce: Invalid address, remove from your list
- Soft bounce: Temporary issue, will retry automatically
- Check bounce reason in **Dashboard > Logs**
