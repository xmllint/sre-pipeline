# Jira API Authentication Reference

## Token Types

Atlassian Cloud has THREE different token types. Only one works for
the Jira REST API.

| Token Type | Generated At | Auth Method | Works for REST API? |
|------------|-------------|-------------|---------------------|
| **Classic API token** | https://id.atlassian.com/manage-profile/security/api-tokens | Basic: `email:token` base64 | ✅ Yes |
| **OAuth app token (scoped)** | https://developer.atlassian.com/console/ | OAuth 2.0 Bearer | ❌ No — needs 3LO flow for access token |
| **Admin API key (unscoped)** | Atlassian Admin → API keys | Varies | ❌ No — admin APIs only |

## Auth Header Format

Classic API token (the one that works):

```
Authorization: Basic base64(email:token)
```

Example:
```
user@example.com:ATATT3x...
→ base64 → dXNlckBleGFtcGxlLmNvbTpBVEFUVDN4Li4u
→ Authorization: Basic dXNlckBleGFtcGxlLmNvbTpBVEFUVDN4Li4u
```

## Prerequisites

1. **Token exists:** https://id.atlassian.com/manage-profile/security/api-tokens → "Create API token"
2. **Site-level API access enabled:** Site admin must toggle "Allow API tokens" at https://<site>.atlassian.net/admin
3. **Project permissions:** User must have "Browse Projects" and "Create Issues" in the target project

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `401 Unauthorized` | Token expired or API access disabled at site level | Check site admin toggle; regenerate token |
| `403 Forbidden` | OAuth token used instead of classic API token | Use classic API token from profile page |
| `The target project doesn't exist` (400) | Token has site access but no project permission | Check project permission scheme |
| `Failed to parse Connect Session Auth Token` | Bearer token sent to REST API | Use Basic auth, not Bearer |
| `Client must be authenticated` | Token format wrong or API disabled | Verify `email:token` format; check site toggle |

## Testing Auth

Minimal curl to verify:

```bash
curl -s -H "Authorization: Basic $(echo -n 'email@example.com:TOKEN' | base64)" \
  "https://yoursite.atlassian.net/rest/api/3/myself"
```

Expected: JSON with `displayName`, `emailAddress`, `accountId`.

## jira-cli Configuration

```yaml
# ~/.config/.jira/.config.yml
server: https://yoursite.atlassian.net
login: email@example.com
project: PROJ
auth_type: basic
installation: cloud
```

Then: `export JIRA_API_TOKEN="email:token"` before running jira-cli commands.

jira-cli's `init` command may fail with 401 even when the token works
for direct API calls. In that case, write the config manually and
skip `jira init`.

## Cloud ID

For `api.atlassian.com` endpoints (rarely needed for Jira REST API):

```bash
curl -s "https://yoursite.atlassian.net/_edge/tenant_info"
# → {"cloudId":"3a5a5dc7-..."}
```

The cloud ID is used with `api.atlassian.com/ex/jira/{cloudId}/...` but
the instance-level API (`yoursite.atlassian.net/rest/api/3/...`) is the
standard path and doesn't need the cloud ID.
