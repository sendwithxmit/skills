---
name: transmit
description: >
  Transmit email API. Send transactional emails, manage templates,
  and sync contacts. Triggers on: send email, transactional email,
  email API, transmit, xmit.
license: MIT
metadata:
  author: sendwithxmit
  version: "1.0"
---

# Transmit Email API

Transmit is a developer-first email platform for transactional email. This skill covers sending emails, creating templates, managing senders, and syncing contacts via the REST API.

## Prerequisites

1. Create an account at https://xmit.sh/sign-up
2. **Add a domain**: Dashboard > Domains > Add Domain. Configure the DNS records shown, then verify.
3. **Create a sender**: Dashboard > Senders > Create. Select your domain and enter the sender email. Copy the **Sender ID** (`snd_xxxxx`) from the row or actions menu.
4. **Generate an API key**: Dashboard > Settings > API Keys > Create. Copy the key immediately (starts with `pm_live_`, shown only once).
5. Set the `TRANSMIT_API_KEY` environment variable with your API key.

## Authentication

```
Authorization: Bearer pm_live_xxxxx
```

Base URL: `https://api.xmit.sh`

---

## Quick Start

```bash
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "from": "hello@yourdomain.com",
    "subject": "Hello!",
    "html": "<h1>Welcome</h1><p>Thanks for signing up.</p>"
  }'
```

Response:

```json
{
  "messageId": "msg_xxxxx",
  "status": "sent"
}
```

---

## Send Email

### POST /email/send

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to` | string or string[] | Yes | Recipient email(s), max 50 total (to + cc + bcc) |
| `subject` | string | Yes | Subject line |
| `html` | string | If no templateId | HTML body |
| `from` | string | If no senderId | Sender email (e.g., `Name <hello@example.com>`) |
| `senderId` | string | If no from | Pre-configured sender ID |
| `fromName` | string | No | Display name (used with `from`) |
| `text` | string | No | Plain text body (auto-generated from HTML if omitted) |
| `cc` | string or string[] | No | CC recipients |
| `bcc` | string or string[] | No | BCC recipients |
| `replyTo` | string | No | Reply-To address |
| `headers` | object | No | Custom email headers |
| `inReplyTo` | string | No | Message-ID for threading |
| `references` | string | No | Thread Message-IDs (space-separated) |
| `attachments` | array | No | File attachments (max 5MB each, 7MB total) |
| `templateId` | string | No | Template ID (replaces html/text) |
| `variables` | object | No | Template variable substitutions |
| `metadata` | object | No | Custom key-value data |

### Send with a Template

```bash
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "senderId": "snd_xxxxx",
    "subject": "Your order shipped",
    "templateId": "tpl_xxxxx",
    "variables": {
      "name": "Alice",
      "trackingUrl": "https://example.com/track/123"
    }
  }'
```

### Send with Attachments

```bash
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "from": "billing@yourdomain.com",
    "subject": "Your Invoice",
    "html": "<p>Please find your invoice attached.</p>",
    "attachments": [
      {
        "filename": "invoice.pdf",
        "content": "<base64-encoded-content>",
        "contentType": "application/pdf"
      }
    ]
  }'
```

### Email Threading

```bash
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "from": "support@yourdomain.com",
    "subject": "Re: Your support request",
    "html": "<p>We have resolved your issue.</p>",
    "inReplyTo": "<original-message-id@example.com>",
    "references": "<original-message-id@example.com>"
  }'
```

---

## Senders

Manage verified sender identities.

### POST /api/senders

```bash
curl https://api.xmit.sh/api/senders \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "domainId": "dom_xxxxx",
    "email": "support@yourdomain.com",
    "name": "Support Team",
    "replyTo": "replies@yourdomain.com"
  }'
```

`domainId` and `email` are required. The email must match the domain (e.g., `support@yourdomain.com` for domain `yourdomain.com`). Copy the Domain ID from Dashboard > Domains, or from the response when creating a domain.

The response includes the sender `id` (e.g., `snd_xxxxx`). You can also copy it from Dashboard > Senders.

### GET /api/senders

List all senders.

### PUT /api/senders/:id

Update sender name or replyTo.

### DELETE /api/senders/:id

Delete a sender.

---

## Templates

Create reusable email templates with `{{variable}}` substitution.

### POST /api/templates

```bash
curl https://api.xmit.sh/api/templates \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Order Confirmation",
    "subject": "Order #{{orderId}} confirmed",
    "bodyHtml": "<h1>Thanks, {{name}}!</h1><p>Your order #{{orderId}} is confirmed.</p>"
  }'
```

`name`, `subject`, and `bodyHtml` are required. Optionally include `bodyText` for a plain text version.

The response includes the template `id` (e.g., `tpl_xxxxx`). Use this when sending emails with `templateId`. You can also copy it from Dashboard > Templates.

### GET /api/templates

List templates. Supports `q` for search, `limit` and `offset` for pagination.

### GET /api/templates/:id

Get template details.

### PUT /api/templates/:id

Update template content.

### POST /api/templates/:id/preview

Preview a rendered template with sample variables.

```bash
curl https://api.xmit.sh/api/templates/tpl_xxxxx/preview \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "variables": { "name": "Alice", "orderId": "12345" } }'
```

### DELETE /api/templates/:id

Delete a template.

---

## Contacts

Sync contact records from your application.

### POST /api/contacts

```bash
curl https://api.xmit.sh/api/contacts \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice@example.com",
    "firstName": "Alice",
    "lastName": "Smith",
    "metadata": { "plan": "pro" }
  }'
```

### POST /api/contacts/bulk

Add up to 25 contacts at once.

```bash
curl https://api.xmit.sh/api/contacts/bulk \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contacts": [
      { "email": "alice@example.com", "firstName": "Alice" },
      { "email": "bob@example.com", "firstName": "Bob" }
    ]
  }'
```

### GET /api/contacts

List contacts. Supports `q`, `limit`, `offset`.

### GET /api/contacts/:id

Get contact details.

### PUT /api/contacts/:id

Update contact fields.

### POST /api/contacts/:id/unsubscribe

Mark a contact as unsubscribed.

### DELETE /api/contacts/:id

Delete a contact.

### POST /api/contacts/bulk-delete

Delete multiple contacts at once (max 500).

```bash
curl https://api.xmit.sh/api/contacts/bulk-delete \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "ids": ["con_xxxxx", "con_yyyyy"] }'
```

### POST /api/contacts/import

Bulk import contacts with full metadata support. Handles deduplication automatically. Optionally add all imported contacts to a list.

```bash
curl https://api.xmit.sh/api/contacts/import \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contacts": [
      { "email": "alice@example.com", "firstName": "Alice", "metadata": { "plan": "pro" } },
      { "email": "bob@example.com", "firstName": "Bob" }
    ],
    "listId": "lst_xxxxx"
  }'
```

---

## Error Handling

All errors return JSON:

```json
{
  "error": "Error description"
}
```

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Invalid request (missing fields, bad format) |
| 401 | Invalid or missing API key |
| 403 | Feature not available on your plan, or limit exceeded |
| 404 | Resource not found |
| 409 | Duplicate resource (e.g., contact or sender already exists) |
| 429 | Rate limited (back off and retry with exponential delay) |
| 500 | Server error |

**Pagination:** List endpoints accept `limit` (default 50, max 1000) and `offset`:

```bash
curl "https://api.xmit.sh/api/contacts?limit=25&offset=50" \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```
