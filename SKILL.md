---
name: transmit
description: >
  Transmit email API. Send transactional emails, manage templates, campaigns,
  lists, contacts, and suppressions. Triggers on: send email, transactional email,
  email API, campaign, newsletter, transmit, xmit.
license: MIT
metadata:
  author: sendwithxmit
  version: "2.0"
---

# Transmit Email API

Transmit is a developer-first email platform for transactional and marketing email. This skill covers sending emails, managing templates, campaigns, lists, contacts, suppressions, and senders via the REST API.

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

`domainId` and `email` are required. The email must match the domain. The response includes the sender `id` (e.g., `snd_xxxxx`).

### GET /api/senders

List all senders. Returns `{ senders: [...] }`.

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

`name`, `subject`, and `bodyHtml` are required. Optionally include `bodyText` and `variables` (array of expected variable names).

### GET /api/templates

List templates. Supports `q` for search, `limit` and `offset` for pagination.

### GET /api/templates/:id

Get template details including full HTML body.

### PUT /api/templates/:id

Update template content. Send only fields to change.

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

## Campaigns

Create and manage email campaigns. All campaigns start as drafts. Requires a marketing plan.

### POST /api/campaigns

```bash
curl https://api.xmit.sh/api/campaigns \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "March Newsletter",
    "subject": "What is new this month",
    "bodyHtml": "<h1>March Update</h1><p>Here is what happened...</p>",
    "senderId": "snd_xxxxx",
    "listId": "lst_xxxxx"
  }'
```

Required: `name`, `subject`, `bodyHtml`. Optional: `senderId`, `listId`, `templateId`, `scheduledAt` (ISO 8601), `isPublic`.

Response: `{ "id": "cmp_xxxxx", "status": "draft" }`

### GET /api/campaigns

List campaigns with stats. Supports `q`, `limit`, `offset`. Returns campaign name, status, and delivery stats (sent, opens, clicks, bounces, complaints).

### GET /api/campaigns/:id

Get full campaign details including HTML body and stats.

### PUT /api/campaigns/:id

Update a draft campaign. Only draft campaigns can be edited.

### POST /api/campaigns/:id/send

Queue or schedule a campaign for sending. Campaign must be in draft status with a listId assigned.

### POST /api/campaigns/:id/pause

Pause a campaign that is currently sending.

### POST /api/campaigns/:id/clone

Duplicate a campaign as a new draft. Appends "(Copy)" to the name.

### POST /api/campaigns/:id/publish

Publish a sent campaign to the public archive.

### POST /api/campaigns/:id/unpublish

Remove a campaign from the public archive.

### DELETE /api/campaigns/:id

Delete a campaign. Cannot delete campaigns currently sending.

---

## Lists

Manage contact lists for campaign targeting. Requires a marketing plan.

### POST /api/lists

```bash
curl https://api.xmit.sh/api/lists \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Newsletter Subscribers",
    "description": "Users who opted in to the monthly newsletter"
  }'
```

### GET /api/lists

List all lists with contact counts. Supports `q`, `limit`, `offset`.

### GET /api/lists/:id

Get list details with paginated members. Supports `q`, `limit`, `offset` for member search.

### PUT /api/lists/:id

Update list name or description.

### POST /api/lists/:id/contacts

Add contacts to a list. Body: `{ "contactIds": ["con_xxxxx", "con_yyyyy"] }`. Skips duplicates.

### DELETE /api/lists/:id/contacts

Remove contacts from a list. Body: `{ "contactIds": ["con_xxxxx"] }`.

### DELETE /api/lists/:id

Delete a list.

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

### PUT /api/contacts/:id

Update contact fields.

### POST /api/contacts/:id/unsubscribe

Mark a contact as unsubscribed. Also exits them from any active sequences.

### DELETE /api/contacts/:id

Delete a contact.

### POST /api/contacts/bulk-delete

Delete multiple contacts at once. Body: `{ "ids": ["con_xxxxx", "con_yyyyy"] }` or `{ "deleteAll": true }`.

### POST /api/contacts/import

Bulk import with deduplication. Optionally add to a list.

```bash
curl https://api.xmit.sh/api/contacts/import \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contacts": [
      { "email": "alice@example.com", "firstName": "Alice" },
      { "email": "bob@example.com", "firstName": "Bob" }
    ],
    "listId": "lst_xxxxx"
  }'
```

---

## Suppressions

Manage suppressed email addresses (bounces, complaints, unsubscribes).

### GET /api/suppressions

List suppressed emails. Supports `limit`, `offset`. Returns email, reason, source, and created date.

### GET /api/suppressions/check/:email

Check if an email is suppressed. Returns `{ "suppressed": true/false, "reason": "bounce" }`.

### POST /api/suppressions

Add an email to the suppression list.

```bash
curl https://api.xmit.sh/api/suppressions \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "reason": "manual"
  }'
```

Reason must be one of: `unsubscribe`, `bounce`, `complaint`, `manual`.

### DELETE /api/suppressions/:id

Remove from suppression list (by suppression ID or email address).

---

## Messages

View sent message history and delivery events.

### GET /api/messages

List sent messages. Supports `limit`, `offset`. Returns message ID, recipient, subject, status, and timestamps.

### GET /api/messages/:id

Get message details with delivery events (sent, delivered, opened, clicked, bounced, complained).

---

## Reference Guides

For detailed examples, curl commands, and common patterns, see the `references/` directory:

- **api-sending-guide.md** - Sending emails, templates, senders, attachments, threading
- **contacts-guide.md** - Contact management, bulk operations, sync patterns
- **campaigns-guide.md** - Campaign lifecycle, scheduling, cloning, archive publishing
- **lists-guide.md** - Contact list management, adding/removing members
- **suppressions-guide.md** - Suppression management, checking addresses, re-enabling

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
