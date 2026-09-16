# Pingram AI Skills

AI-powered skills to help developers integrate Pingram without relying on documentation.

## What is Pingram?

Pingram is a unified communications platform for developers. It allows you to send and receive emails, SMS, and manage voice calls through a simple API. Pingram is designed for fast integration with built-in telephony management, email deliverability management, multi-channel delivery; all backed by API-first and AI-powered developer tools and rich logs and analytics.

## Installation

Install all Pingram skills with:

```bash
npx skills add pingram-io/skills
```

## Available Skills

| Skill                                                         | Description                                                 |
| ------------------------------------------------------------- | ----------------------------------------------------------- |
| [pingram-getting-started](./pingram-getting-started/SKILL.md) | Set up Pingram and send your first notification             |
| [pingram-email](./pingram-email/SKILL.md)                     | Email configuration: domains, inbound, outbound, SMTP relay |
| [pingram-sms](./pingram-sms/SKILL.md)                         | SMS notification configuration and delivery                 |
| [pingram-voice](./pingram-voice/SKILL.md)                     | Voice call notifications with text-to-speech                |
| [pingram-inbound](./pingram-inbound/SKILL.md)                 | Receive inbound emails and SMS (SMS requires paid account)  |
| [pingram-webhooks](./pingram-webhooks/SKILL.md)               | Event webhooks for delivery tracking                        |
| [pingram-broadcasts](./pingram-broadcasts/SKILL.md)           | Email broadcasts: campaigns, audiences, scheduling, metrics |
| [pingram-users](./pingram-users/SKILL.md)                     | User identification: identify, custom properties, audiences |

## What are Skills?

Skills are AI-readable documentation files that help coding assistants understand how to use APIs and libraries. When you install these skills, your AI assistant can help you:

- Set up Pingram from scratch
- Send notifications across multiple channels
- Configure email domains and SMTP
- Handle inbound messages
- Track delivery with webhooks

## Learn More

- [Pingram Documentation](https://www.pingram.io/docs/)
- [Skills CLI](https://npmjs.com/package/skills)
