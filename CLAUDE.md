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

### Attaching Files to Tickets
```bash
curl -s -X POST \
  -u "email:api_token" \
  -H "X-Atlassian-Token: no-check" \
  -F "file=@/path/to/file.png;filename=descriptive_name.png" \
  "https://kobumura.atlassian.net/rest/api/3/issue/XXX-123/attachments"
```
Use `;filename=` to rename files to descriptive names during upload.

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

## Session Handoffs

Template: `shared/session-handoff-template.md`

Create in each project's `docs/session_handoffs/` directory when:
- Approaching token/context limits
- At natural stopping points
- Before switching to different tasks
