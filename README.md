# LittleTalks Documentation

Shared documentation hub for the LittleTalks ecosystem.

## Why This Exists

LittleTalks is built across **multiple projects in different languages** that share significant overlap:

| Project | Language | Purpose |
|---------|----------|---------|
| **littletalks-mobile** | React Native / JavaScript | iOS & Android mobile app |
| **littletalks-api** | Node.js / Express | Backend API on Heroku |
| **littletalks-admin** | PHP | Admin dashboard & analytics |
| **littlepipes** | GitHub Actions / YAML | End-to-end CI/CD platform |

These projects share:
- **Business logic** - User journeys, subscription tiers, analytics events
- **Integrations** - RevenueCat, Twilio, OneSignal, Google Analytics
- **Standards** - Jira workflows, commit guidelines, session handoffs
- **Data contracts** - API endpoints, entitlement flows

Rather than duplicate documentation across repos (and watch it drift out of sync), this repo serves as the **single source of truth** for cross-project knowledge.

## Structure

```
littletalks-docs/
├── CLAUDE.md                          # Mother CLAUDE - shared context for all projects
├── DOCUMENTATION-ARCHITECTURE.md      # How this documentation system works
├── README.md                          # This file
├── shared/                            # Cross-project technical documentation
│   ├── journey-system.md              # User journey architecture
│   ├── analytics-events.md            # Event tracking system
│   ├── revenuecat-integration.md      # Subscription/entitlement flow
│   ├── api-contracts.md               # API endpoints
│   └── session-handoff-template.md    # Template for session continuity
├── littlepipes/                       # CI/CD platform docs (private - littlepipes repo is public)
│   └── [build system documentation]
└── articles/                          # Write-ups and guides
    └── claude-documentation-system.md # How we built this system
```

## Related Repositories

### Kobumura (LittleTalks Ecosystem)
- [littletalks-mobile](https://github.com/Kobumura/littletalks-mobile) - React Native app
- [littletalks-admin](https://github.com/Kobumura/littletalks-admin) - PHP admin dashboard
- [littletalks-api](https://github.com/Kobumura/littletalks-api) - Node.js backend API
- [littlepipes](https://github.com/Kobumura/littlepipes) - LittlePipes CI/CD (public repo)

### Personal Projects (use shared standards)
- [football](https://github.com/dorothyjb/football) - Football pool greenfield rewrite
- [football-pool-legacy](https://github.com/dorothyjb/football-pool-legacy) - Legacy archive (2011-2020)
- [WXING](https://github.com/dorothyjb/WXING) - Reference patterns (JiraService, notifications)

## How It Works

Each project's `CLAUDE.md` references this repo for shared standards, while containing project-specific details (tech stack, file structure, dev commands). Deep technical docs live here and are read on-demand.

See `DOCUMENTATION-ARCHITECTURE.md` for the full explanation.
