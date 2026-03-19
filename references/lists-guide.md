# Lists Guide

Manage contact lists for campaign targeting. Lists require a marketing plan.

## Create a List

```bash
curl https://api.xmit.sh/api/lists \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Newsletter Subscribers",
    "description": "Users who opted in to the monthly newsletter"
  }'
```

Response:

```json
{ "id": "lst_xxxxx" }
```

## List All Lists

```bash
curl "https://api.xmit.sh/api/lists?limit=50&offset=0" \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Returns lists with contact counts. Supports `q` for search.

## Get List Details

Returns list info with paginated members.

```bash
curl "https://api.xmit.sh/api/lists/lst_xxxxx?limit=25&offset=0" \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Members include email, firstName, lastName, status, and createdAt.

## Update a List

```bash
curl -X PUT https://api.xmit.sh/api/lists/lst_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "VIP Subscribers",
    "description": "High-value customers"
  }'
```

## Add Contacts to a List

Pass an array of contact IDs. Duplicates are skipped.

```bash
curl -X POST https://api.xmit.sh/api/lists/lst_xxxxx/contacts \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contactIds": ["con_xxxxx", "con_yyyyy", "con_zzzzz"]
  }'
```

Response:

```json
{ "added": 3 }
```

## Remove Contacts from a List

```bash
curl -X DELETE https://api.xmit.sh/api/lists/lst_xxxxx/contacts \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contactIds": ["con_xxxxx"]
  }'
```

Response:

```json
{ "removed": 1 }
```

## Delete a List

```bash
curl -X DELETE https://api.xmit.sh/api/lists/lst_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Common Patterns

### Import Contacts Directly into a List

Use the contacts import endpoint with a `listId`:

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

### Create a List and Use It for a Campaign

```bash
# 1. Create the list
curl https://api.xmit.sh/api/lists \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "Launch Announcement" }'

# 2. Add contacts
curl -X POST https://api.xmit.sh/api/lists/lst_xxxxx/contacts \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "contactIds": ["con_aaa", "con_bbb", "con_ccc"] }'

# 3. Create a campaign targeting this list
curl https://api.xmit.sh/api/campaigns \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Launch Announcement",
    "subject": "We just launched!",
    "bodyHtml": "<h1>Big news</h1>",
    "senderId": "snd_xxxxx",
    "listId": "lst_xxxxx"
  }'
```
