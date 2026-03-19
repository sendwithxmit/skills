# Suppressions Guide

Manage suppressed email addresses. Suppressions prevent emails from being sent to addresses that have bounced, complained, or unsubscribed.

## List Suppressions

```bash
curl "https://api.xmit.sh/api/suppressions?limit=100&offset=0" \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Returns suppressed emails with reason, source, and date. Only active (non-expired) suppressions are returned.

## Check If an Email Is Suppressed

```bash
curl https://api.xmit.sh/api/suppressions/check/user@example.com \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Response when suppressed:

```json
{
  "suppressed": true,
  "reason": "bounce",
  "createdAt": "2026-01-15T12:00:00Z"
}
```

Response when not suppressed:

```json
{ "suppressed": false }
```

## Add to Suppression List

Manually suppress an email address.

```bash
curl https://api.xmit.sh/api/suppressions \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "reason": "manual",
    "source": "api"
  }'
```

Reason must be one of: `unsubscribe`, `bounce`, `complaint`, `manual`. Source is optional (defaults to `"api"`).

Returns 409 if the email is already suppressed.

## Remove from Suppression List

Remove by suppression ID:

```bash
curl -X DELETE https://api.xmit.sh/api/suppressions/sup_xxxxx \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Or remove by email address:

```bash
curl -X DELETE https://api.xmit.sh/api/suppressions/user@example.com \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```

Both work. Silently succeeds if the suppression does not exist.

## How Suppressions Work

Suppressions are created automatically when:
- An email **bounces** (hard bounce)
- A recipient files a **complaint** (marks as spam)
- A contact **unsubscribes** via the unsubscribe link

You can also add suppressions manually via the API or dashboard.

When sending an email, Transmit checks the suppression list and skips suppressed addresses. This protects your sender reputation.

## Common Patterns

### Check Before Sending

```bash
# Check suppression status
curl https://api.xmit.sh/api/suppressions/check/user@example.com \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"

# If not suppressed, send the email
curl https://api.xmit.sh/email/send \
  -H "Authorization: Bearer $TRANSMIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "senderId": "snd_xxxxx",
    "subject": "Your report",
    "html": "<p>Here is your weekly report.</p>"
  }'
```

### Re-enable a Suppressed Address

If a user wants to receive emails again after being suppressed:

```bash
# Remove the suppression
curl -X DELETE https://api.xmit.sh/api/suppressions/user@example.com \
  -H "Authorization: Bearer $TRANSMIT_API_KEY"
```
