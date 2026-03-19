# Campaigns Guide

Create and manage email campaigns via the REST API. Campaigns require a marketing plan.

## Create a Campaign

All campaigns start as drafts.

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

Required: `name`, `subject`, `bodyHtml`. Optional: `senderId`, `listId`, `templateId`, `scheduledAt` (ISO 8601), `isPublic` (boolean).

Response:

```json
{ "id": "cmp_xxxxx", "status": "draft" }
```

## List Campaigns

```bash
curl "https://api.xmit.sh/api/campaigns?limit=25&offset=0&q=newsletter" \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Returns campaigns with delivery stats (sent, opens, clicks, bounces, complaints).

## Get Campaign Details

```bash
curl https://api.xmit.sh/api/campaigns/cmp_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Returns full campaign including HTML body and stats.

## Update a Draft Campaign

Only campaigns in `draft` status can be edited.

```bash
curl -X PUT https://api.xmit.sh/api/campaigns/cmp_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "Updated subject line",
    "senderId": "snd_yyyyy"
  }'
```

## Send a Campaign

Queue a draft for immediate sending or schedule it for later. The campaign must have a `listId` assigned.

```bash
curl -X POST https://api.xmit.sh/api/campaigns/cmp_xxxxx/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

If the campaign has a `scheduledAt` date in the future, it will be scheduled. Otherwise it sends immediately.

## Pause a Sending Campaign

```bash
curl -X POST https://api.xmit.sh/api/campaigns/cmp_xxxxx/pause \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Duplicate a Campaign

Creates a copy as a new draft with "(Copy)" appended to the name.

```bash
curl -X POST https://api.xmit.sh/api/campaigns/cmp_xxxxx/clone \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Publish to Archive

Make a sent campaign publicly visible on your archive subdomain.

```bash
curl -X POST https://api.xmit.sh/api/campaigns/cmp_xxxxx/publish \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Remove from Archive

```bash
curl -X POST https://api.xmit.sh/api/campaigns/cmp_xxxxx/unpublish \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Delete a Campaign

Cannot delete campaigns that are currently sending.

```bash
curl -X DELETE https://api.xmit.sh/api/campaigns/cmp_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Campaign Status Flow

```
draft -> queued -> sending -> sent
draft -> scheduled -> queued -> sending -> sent
sending -> paused -> sending (resume by re-sending)
```

## Common Patterns

### Create and Schedule a Campaign

```bash
# 1. Create the draft
curl https://api.xmit.sh/api/campaigns \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Product Launch",
    "subject": "Introducing our new feature",
    "bodyHtml": "<h1>Big news!</h1><p>We just launched...</p>",
    "senderId": "snd_xxxxx",
    "listId": "lst_xxxxx",
    "scheduledAt": "2026-04-01T10:00:00Z"
  }'

# 2. Queue it for scheduled sending
curl -X POST https://api.xmit.sh/api/campaigns/cmp_xxxxx/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

### Clone and Iterate

```bash
# Clone an existing campaign
curl -X POST https://api.xmit.sh/api/campaigns/cmp_xxxxx/clone \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"

# Update the clone
curl -X PUT https://api.xmit.sh/api/campaigns/cmp_yyyyy \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "April Newsletter",
    "subject": "April updates"
  }'
```
