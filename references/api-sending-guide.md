# Sending Emails with Transmit

Complete guide to sending transactional emails via the REST API.

## Base URL

```
https://api.xmit.sh
```

## Authentication

```
Authorization: Bearer pm_live_xxxxx
```

## POST /email/send

### Minimal Example

```bash
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "from": "hello@yourdomain.com",
    "subject": "Hello!",
    "html": "<h1>Welcome</h1><p>Thanks for joining.</p>"
  }'
```

### All Parameters

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `to` | string or string[] | Yes | One or more recipients |
| `subject` | string | Yes | Subject line |
| `html` | string | If no templateId | HTML body |
| `from` | string | If no senderId | Sender email (e.g., `Name <email@domain.com>`) |
| `senderId` | string | If no from | Use a pre-configured sender |
| `fromName` | string | No | Display name when using `from` |
| `text` | string | No | Plain text (auto-generated if omitted) |
| `cc` | string or string[] | No | CC recipients |
| `bcc` | string or string[] | No | BCC recipients |
| `replyTo` | string | No | Reply-To address |
| `headers` | object | No | Custom headers (e.g., `X-Entity-Ref-ID`) |
| `inReplyTo` | string | No | For email threading |
| `references` | string | No | Thread references (space-separated) |
| `attachments` | array | No | File attachments (see below) |
| `templateId` | string | No | Template to render |
| `variables` | object | No | Template variable values |
| `metadata` | object | No | Custom key-value data |

### Response

```json
{
  "messageId": "msg_xxxxx",
  "status": "sent"
}
```

## Using Templates

Templates support `{{variable}}` syntax for dynamic content.

### Create a Template

```bash
curl https://api.xmit.sh/api/templates \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Order Confirmation",
    "subject": "Order #{{orderId}} confirmed",
    "bodyHtml": "<h1>Thanks, {{name}}!</h1><p>Order #{{orderId}} is confirmed.</p>"
  }'
```

Required fields: `name`, `subject`, `bodyHtml`. Optionally include `bodyText` for plain text.

### Send with Template

```bash
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "customer@example.com",
    "senderId": "snd_xxxxx",
    "subject": "Order confirmed",
    "templateId": "tpl_xxxxx",
    "variables": {
      "name": "Alice",
      "orderId": "12345"
    }
  }'
```

### Preview a Template

```bash
curl https://api.xmit.sh/api/templates/tpl_xxxxx/preview \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "variables": { "name": "Alice", "orderId": "12345" } }'
```

### Update a Template

```bash
curl -X PUT https://api.xmit.sh/api/templates/tpl_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Order Confirmation v2",
    "subject": "Order #{{orderId}} confirmed",
    "bodyHtml": "<h1>Thanks, {{name}}!</h1><p>Your order is on the way.</p>"
  }'
```

## Attachments

Attachments are base64-encoded:

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
        "content": "JVBERi0xLjQK...",
        "contentType": "application/pdf"
      }
    ]
  }'
```

## Senders

### Create a Sender

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

`domainId` and `email` are required. The email must match the domain.

### List Senders

```bash
curl https://api.xmit.sh/api/senders \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

### Set Default Sender

```bash
curl -X PUT https://api.xmit.sh/api/senders/snd_xxxxx/default \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Email Threading

To create email threads (conversation view in recipients' inboxes), use `inReplyTo` and `references`:

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

## Error Handling

| Status | Meaning |
|--------|---------|
| 200 | Email sent successfully |
| 400 | Missing required fields or invalid format |
| 401 | Invalid API key |
| 403 | Plan limit exceeded or feature unavailable |
| 422 | Recipient is suppressed |
| 500 | Server error (retry with backoff) |
