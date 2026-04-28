---
name: pingram-getting-started
description: Set up Pingram and send your first notification. Use when the user wants to integrate Pingram, install the SDK, configure API keys, or send their first notification.
---

# Getting Started with Pingram

Pingram is a notification service for sending Email, SMS, Voice, In-App, Push, and Slack notifications through a single API.

## Quick Start

### 1. Install the SDK

Choose your language:

**Node.js / TypeScript:**

```bash
npm install pingram
```

**Python:**

```bash
pip install pingram-python
```

**Go:**

```bash
go get github.com/notificationapi-com/pingram-go
```

**PHP:**

```bash
composer require pingram/php
```

**Ruby:**

```bash
gem install pingram
```

**Java (Maven):**

```xml
<dependency>
  <groupId>io.pingram</groupId>
  <artifactId>pingram</artifactId>
  <version>0.1.0</version>
</dependency>
```

**C# (.NET):**

```bash
dotnet add package Pingram
```

### 2. Initialize the Client

Get your API key from the Pingram dashboard (Settings > API Keys).

**Node.js:**

```typescript
import { Pingram } from 'pingram';

const client = new Pingram({
  apiKey: 'pingram_sk_...'
});
```

**Python:**

```python
from pingram import Pingram

async with Pingram(api_key="pingram_sk_...") as client:
    # use client
```

**Go:**

```go
import "github.com/notificationapi-com/pingram-go"

client := pingram.NewClient("pingram_sk_...")
```

### 3. Send Your First Notification

No dashboard setup required - notification types are auto-generated on first use.

**Node.js:**

```typescript
await client.send({
  type: 'welcome',
  to: {
    email: 'user@example.com',
    number: '+15005550006'
  },
  email: {
    subject: 'Welcome!',
    html: '<h1>Welcome to our app!</h1>',
    senderName: 'Acme Team',
    senderEmail: 'hello@acme.com'
  },
  sms: {
    message: 'Welcome to Acme! Check your email for next steps.'
  }
});
```

**Python:**

```python
from pingram import SenderPostBody, SenderPostBodyTo

await client.send(sender_post_body=SenderPostBody(
    type="welcome",
    to=SenderPostBodyTo(email="user@example.com", number="+15005550006"),
    email={
        "subject": "Welcome!",
        "html": "<h1>Welcome to our app!</h1>",
        "senderName": "Acme Team",
        "senderEmail": "hello@acme.com"
    },
    sms={"message": "Welcome to Acme! Check your email for next steps."}
))
```

**Note:** The `id` field in `to` is optional. If your system tracks users by ID, include it. Otherwise, `email` or `number` alone is sufficient and will also serve as the user identifier.

## Regions

Pingram supports multiple regions. Set the region when initializing:

```typescript
const client = new Pingram({
  apiKey: 'pingram_sk_...',
  region: 'eu' // 'us' (default), 'eu', or 'ca'
});
```

## Next Steps

- **Define notification types in dashboard:** Optionally pre-configure notification types with templates, channel settings, and more
- **Add more channels:** Include `sms`, `call`, `inapp`, `mobile_push`, `web_push`, or `slack` in your send request
- **Use templates:** Create reusable templates in the dashboard and reference them by ID
- **Set up webhooks:** Receive delivery events via webhooks

## Common Issues

**Debugging tips:**

1. Check the API response for error messages
2. Go to **Dashboard > Logs** to see the full message lifecycle and status

If you don't see your send request in the logs, check for authorization or region errors.

**401 Unauthorized:** Check that your API key is correct and has the required permissions. Ensure region is one of: `us`, `eu`, `ca`.

**Missing recipient:** With the `to` object, include `email` for email notifications, `number` for SMS/voice.
