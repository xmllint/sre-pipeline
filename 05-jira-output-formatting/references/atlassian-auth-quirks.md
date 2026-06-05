# Atlassian Auth Quirks

Captured during Jira API import attempts. Token types, auth methods,
and common failure modes.

## Token Types

Atlassian Cloud has TWO distinct token types, generated from different
places, with different auth methods:

| Token Type | Generated At | Auth Method | Works For |
|------------|-------------|-------------|-----------|
| **API token with scopes** (OAuth) | developer.atlassian.com console | OAuth 2.0 (3LO flow) | App integrations, granular permissions |
| **API token without scopes** (classic) | id.atlassian.com/manage-profile/security/api-tokens | Basic auth (email:token) | CLI tools, scripts, REST API |
| **API key without scopes** | admin.atlassian.com → API keys | Basic auth (email:key) | Full admin API access |

## Auth Methods Per Endpoint

| Endpoint | Accepts Basic | Accepts Bearer | Notes |
|----------|--------------|----------------|-------|
| `instance.atlassian.net/rest/api/3/*` | ✅ (classic token only) | ❌ | Instance-level Jira REST API |
| `api.atlassian.com/ex/jira/{cloudId}/*` | ❌ | ✅ (OAuth token) | Cloud-scoped API |
| `api.atlassian.com/admin/*` | ✅ (admin API key) | ❌ | Admin API |

## Common Failure Modes

1. **"Client must be authenticated" (401) on instance API with Bearer token**
   → You're using an OAuth token on the instance API. Switch to classic API token with Basic auth.

2. **"scope does not match" on cloud API**
   → Your OAuth token doesn't include the right scopes for that endpoint. Check developer console scopes.

3. **"Failed to parse Connect Session Auth Token"**
   → You sent a Bearer token to an endpoint that expects Basic auth (or vice versa).

4. **"The target project doesn't exist" (400) even though it does**
   → Token has no project access. Check site-level "Allow API tokens" setting at admin.atlassian.com.

5. **jira-cli "invalid issue types in config"**
   → jira-cli `init` failed to fetch project metadata. The config file is missing issue type definitions. Re-run `jira init` with correct auth.

6. **jira-cli `init` returns 401 even with correct token**
   → jira-cli init makes a specific API call that may require different permissions than issue CRUD. Try direct curl first to verify the token works.

## jira-cli Config

Config file: `~/.config/.jira/.config.yml`

```yaml
server: https://instance.atlassian.net
login: user@example.com
project: PROJECTKEY
auth_type: basic        # "basic" for classic API token, "bearer" for OAuth
installation: cloud
```

API token via env var: `JIRA_API_TOKEN=email:token`

## Pre-Flight Check Before Any Import

```bash
# 1. Verify auth works
curl -s -o /dev/null -w "%{http_code}" \
  --user "email:token" \
  "https://instance.atlassian.net/rest/api/3/myself"
# Expected: 200

# 2. Verify project access
curl -s --user "email:token" \
  "https://instance.atlassian.net/rest/api/3/project/KEY"
# Expected: JSON with project details, not 404

# 3. Verify issue creation
curl -s -X POST \
  --user "email:token" \
  -H "Content-Type: application/json" \
  -d '{"fields":{"project":{"key":"KEY"},"summary":"test","issuetype":{"name":"Task"}}}' \
  "https://instance.atlassian.net/rest/api/3/issue"
# Expected: 201 with issue key
```
