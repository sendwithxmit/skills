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
- `limit` - Results per page (default 50, max 100)
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

## Delete a Contact

```bash
curl -X DELETE https://api.xmit.sh/api/contacts/con_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
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
