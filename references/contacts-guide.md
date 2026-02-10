# Contacts Guide

Sync and manage contact records from your application.

## Create a Contact

```bash
curl https://api.xmit.sh/api/contacts \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice@example.com",
    "firstName": "Alice",
    "lastName": "Smith",
    "metadata": { "plan": "pro", "signupSource": "landing-page" }
  }'
```

## Bulk Add Contacts (up to 25)

```bash
curl https://api.xmit.sh/api/contacts/bulk \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contacts": [
      { "email": "alice@example.com", "firstName": "Alice" },
      { "email": "bob@example.com", "firstName": "Bob" },
      { "email": "carol@example.com", "firstName": "Carol" }
    ]
  }'
```

## List Contacts

```bash
curl "https://api.xmit.sh/api/contacts?limit=25&offset=0&q=alice" \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Query parameters:
- `q` - Search by name or email
- `limit` - Results per page (default 50, max 1000)
- `offset` - Skip N results

## Get a Contact

```bash
curl https://api.xmit.sh/api/contacts/con_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Update a Contact

```bash
curl -X PUT https://api.xmit.sh/api/contacts/con_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Alice",
    "metadata": { "plan": "enterprise" }
  }'
```

## Unsubscribe a Contact

```bash
curl -X POST https://api.xmit.sh/api/contacts/con_xxxxx/unsubscribe \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Delete a Contact

```bash
curl -X DELETE https://api.xmit.sh/api/contacts/con_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

## Bulk Delete Contacts

Delete up to 500 contacts at once by passing their IDs.

```bash
curl https://api.xmit.sh/api/contacts/bulk-delete \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "ids": ["con_xxxxx", "con_yyyyy"] }'
```

## Import Contacts

Import contacts with full metadata support. Existing emails are skipped automatically. Optionally add all imported contacts to a list.

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

Response:

```json
{ "created": 2, "skipped": 0, "errors": [] }
```

## Common Patterns

### Sync Users from Your App

When a user signs up in your application, create a contact in Transmit so you can send them transactional emails:

```bash
# After user signup in your app
curl https://api.xmit.sh/api/contacts \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "firstName": "New",
    "lastName": "User",
    "metadata": { "userId": "usr_123", "plan": "free" }
  }'
```

### Update Contact on Plan Change

```bash
curl -X PUT https://api.xmit.sh/api/contacts/con_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "metadata": { "plan": "pro", "upgradedAt": "2026-02-10" } }'
```
