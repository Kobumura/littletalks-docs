# LittleTalks Shared Documentation

This is the central documentation hub for the LittleTalks ecosystem (Kobumura org).

> **How this all works**: See `DOCUMENTATION-ARCHITECTURE.md` for the full explanation of the three-tier documentation system and maintenance guidelines.

## Kobumura Projects

| Project | Jira | Path (Lenovo) | Description |
|---------|------|---------------|-------------|
| littletalks-mobile | LTD | `C:\Users\dorot\AndroidStudioProjects\littletalks-mobile` | React Native app |
| littletalks-admin | LTD | `C:\Users\dorot\PhpstormProjects\littletalks-admin` | PHP admin dashboard |
| littletalks-api | LTD | `C:\Users\dorot\StudioProjects\littletalks-api` | Node.js/Express backend |
| lp_test (LittlePipes) | LP | `C:\Users\dorot\AndroidStudioProjects\lp_test` | CI/CD platform |

## Personal Projects (not Kobumura, but use shared standards)

| Project | Jira | Path (Lenovo) | Purpose |
|---------|------|---------------|---------|
| WXING | CD | `C:\Users\dorot\PhpstormProjects\WXING` | JiraService, notification system, audit logging |
| football | FOOT | `C:\Users\dorot\PhpstormProjects\football` | Football pool greenfield rewrite |
| football-pool-legacy | - | `C:\Users\dorot\PhpstormProjects\football-pool-legacy` | Reference archive (2011-2020 code) |

## Jira Integration

- **Instance**: `kobumura.atlassian.net`
- **Auth**: Basic auth with email + API token
- **Credentials**: Stored in environment variables (never commit to git!)

### Credential Setup

Add to your shell profile (`~/.bashrc`, PowerShell `$PROFILE`, etc.):

```bash
# Jira API credentials (kobumura.atlassian.net)
export JIRA_EMAIL="dorothyjbt+claude@gmail.com"
export JIRA_TOKEN="your-api-token-here"
```

On Windows PowerShell, add to your `$PROFILE`:
```powershell
$env:JIRA_EMAIL = "dorothyjbt+claude@gmail.com"
$env:JIRA_TOKEN = "your-api-token-here"
```

### Creating Tickets

```bash
# Create a ticket (replace PROJECT with: LTD, LP, CD, or FOOT)
curl -s -X POST \
  -u "$JIRA_EMAIL:$JIRA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "project": {"key": "PROJECT"},
      "summary": "Ticket title here",
      "description": {
        "type": "doc",
        "version": 1,
        "content": [{"type": "paragraph", "content": [{"type": "text", "text": "Description here"}]}]
      },
      "issuetype": {"name": "Task"}
    }
  }' \
  "https://kobumura.atlassian.net/rest/api/3/issue"
```

**Issue types**: `Task`, `Bug`, `Story`, `Epic`

**Response** includes `key` (e.g., "FOOT-42") and `id` for the created ticket.

### Attaching Files to Tickets

```bash
curl -s -X POST \
  -u "$JIRA_EMAIL:$JIRA_TOKEN" \
  -H "X-Atlassian-Token: no-check" \
  -F "file=@/path/to/file.png;filename=descriptive_name.png" \
  "https://kobumura.atlassian.net/rest/api/3/issue/XXX-123/attachments"
```
Use `;filename=` to rename files to descriptive names during upload.

### Adding Comments

```bash
curl -s -X POST \
  -u "$JIRA_EMAIL:$JIRA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "body": {
      "type": "doc",
      "version": 1,
      "content": [{"type": "paragraph", "content": [{"type": "text", "text": "Comment text here"}]}]
    }
  }' \
  "https://kobumura.atlassian.net/rest/api/3/issue/XXX-123/comment"
```

## User Context (Dorothy)

### Screenshot Locations
- **Windows**: `C:\Users\dorot\Downloads\`
- **MacBook**: `~/Desktop/`

### Commit & PR Guidelines
- Do NOT include "Generated with Claude Code", "Co-Authored-By: Claude", or any AI attribution

### Platform Detection
- `win32` + `dorot` in path = Lenovo (Primary PC)
- `darwin` = MacBook (be patient - Dorothy is learning Mac!)

## Documentation Locations

| Type | Location | When to Use |
|------|----------|-------------|
| Business docs | Confluence | Marketing, high-level strategy |
| Shared technical | `shared/` in this repo | Cross-project guides (journey, analytics, RevenueCat) |
| CI/CD platform | `littlepipes/` in this repo | Build system docs (private - lp_test is public!) |
| Project-specific | Each repo's CLAUDE.md | Tech stack, file structure, dev commands |
| Session history | Each repo's `docs/session_handoffs/` | Continuity between sessions |

## Key Shared Technical Docs

**IMPORTANT**: Read these docs when working on relevant projects!

| Doc | Projects | Content |
|-----|----------|---------|
| `shared/php-data-access.md` | littletalks-admin, WXING, football | **Data access layer** - dynamic SQL builders for audit-safe persistence, repository pattern |
| `shared/php-migrations.md` | littletalks-admin, WXING, football | **Database migrations** - how to create SQL and PHP migrations, auto-deploy integration |
| `shared/journey-system.md` | littletalks-mobile, littletalks-api | User journey/onboarding system |
| `shared/analytics-events.md` | littletalks-mobile, littletalks-api | Analytics event taxonomy |
| `shared/revenuecat-integration.md` | littletalks-mobile | Subscription/paywall setup |
| `shared/api-contracts.md` | littletalks-mobile, littletalks-api | API endpoint specifications |

## Session Handoffs

Template: `shared/session-handoff-template.md`

Create in each project's `docs/session_handoffs/` directory when:
- Approaching token/context limits
- At natural stopping points
- Before switching to different tasks

## Security: Protect docs/ Folders

For PHP projects deployed to web servers, add `.htaccess` to `docs/` to prevent web access:

```apache
# Deny all web access to this directory
Require all denied
```

**Applies to**: littletalks-admin, football, WXING (PHP on Plesk/Apache)
**Does NOT apply to**: littletalks-mobile (React Native), littletalks-api (Node.js), lp_test (GitHub Actions)
