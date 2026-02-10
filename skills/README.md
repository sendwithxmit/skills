# Transmit Skills.sh Skill

Agent skill for [Transmit](https://xmit.sh), a developer-first email platform.

## What This Skill Does

Gives AI agents the ability to:

- Send transactional emails via REST API
- Create and manage email templates
- Set up sender identities
- Sync contacts from your application

## Install

```bash
npx skills add sendwithxmit/skills
```

## Getting Started

1. Sign up at [xmit.sh/sign-up](https://xmit.sh/sign-up)
2. Add and verify a domain (Dashboard > Domains)
3. Create a sender identity (Dashboard > Senders)
4. Generate an API key (Dashboard > Settings > API Keys)
5. Set `TRANSMIT_API_KEY` in your environment

## Quick Start

```bash
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "from": "hello@yourdomain.com",
    "subject": "Hello!",
    "html": "<h1>Welcome</h1>"
  }'
```

## Reference Guides

See the `references/` directory for detailed guides:

- **api-sending-guide.md** - Sending emails, templates, senders, attachments, threading
- **contacts-guide.md** - Contact management and sync patterns

## Links

- [Transmit Website](https://xmit.sh)
- [API Documentation](https://xmit.sh/docs/api)
- [Dashboard](https://xmit.sh/dashboard)
