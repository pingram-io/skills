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
      // false skips that kind of tracking for this send. Delivery events still record.
      openTracking: false,
      clickTracking: false,
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

Sending and receiving use different limits.

- **Inline (`content`):** ~4 MB raw per attachment (~6 MB total request payload after base64 and JSON overhead). Returns 413 when exceeded. Works via API and SMTP. Several inline files share that request budget.
- **URL (`url`):** Up to 20 MB per attachment. Pingram fetches the file at send time. API only — not available via SMTP.
- For files over ~4 MB, use a URL attachment via the API.
- **Inbound:** The whole message, including headers, can be up to 40 MB. There is no separate per-attachment cap. Larger mail is bounced before Pingram stores it. Attachments are base64 in the webhook JSON, so the HTTP body is about one third larger than the raw message. The endpoint must accept that JSON. See the pingram-inbound skill.

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

### Your domain

Verify the domain, add the inbound MX record shown in **Settings > Domains**, and subscribe a webhook to `EMAIL_INBOUND`. Every address on that domain and its subdomains is delivered after DKIM succeeds. Adding a domain before DKIM succeeds does not deliver its mail. Saved addresses are not required.

### Trying inbound without a domain

Each account has a Pingram address such as `yourcompany@mail.pingram.io`. Use it to try inbound email before you connect a domain. Do not use it for production traffic.

### Inbound Webhook Payload

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
  "subject": "Help needed",
  "bodyText": "Plain text body",
  "bodyHtml": "<p>HTML body</p>",
  "attachments": [
    {
      "filename": "document.pdf",
      "contentType": "application/pdf",
      "size": 12345,
      "content": "base64-encoded-content",
      "contentId": "document",
      "contentDisposition": "attachment"
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

`matched` is the address that received the mail, including any address on your domain or its subdomains, and when that address was only in Cc or Bcc. Compare `matched.email` with `toRecipients` and `ccRecipients` to see which header contained it. `toRecipients` and `ccRecipients` are the headers with display names. `to` and `cc` are deprecated and unchanged for existing webhooks. Use `toRecipients` and `ccRecipients`. `receivedAt` is when the message was accepted. `sentAt` is the sender Date header and is omitted when that header is missing or invalid. `fromName` is the sender display name. `contentId` matches `cid:` in the HTML body. Inbound size limits are in [Size limits](#size-limits) above. One inbound email counts as 1 email. Attachment size does not add usage. Resend a failed inbound webhook for 30 days with `logs.retryInbound`. A resend does not count as another email.

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
- Or call `logs.getDeliveryStatus` (`GET /logs/status/{trackingIds}`) for email and SMS. It returns the tracking id, channel, status, and timestamp only
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

**413 Payload Too Large:**

- An inline attachment is over ~4 MB raw, or several inline attachments together exceed the request budget
- Use a URL attachment for a file up to 20 MB
- Inbound mail uses the 40 MB whole-message limit in [Size limits](#size-limits)
