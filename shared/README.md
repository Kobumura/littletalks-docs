# Shared Technical Documentation

Cross-project guides that apply to multiple LittleTalks repositories.

## Available Documents

| Document | Description | Used By |
|----------|-------------|---------|
| `journey-system.md` | Intelligent journey system architecture, database schema, backend/frontend implementation | mobile, admin |
| `analytics-events.md` | Comprehensive analytics event tracking system | mobile, admin |
| `revenuecat-integration.md` | RevenueCat entitlement flow and subscription handling | mobile, admin, api |
| `api-contracts.md` | API endpoints and entitlement flow | mobile, admin, api |
| `session-handoff-template.md` | Template for session continuity handoffs | all projects |
| `php-migrations.md` | SQL migration system for PHP projects with Plesk/GitHub Actions | admin, WXING, football |
| `deploy-workflow-template.yml` | GitHub Actions deploy workflow template for PHP/Plesk projects | admin, WXING, football |

## Usage

Project CLAUDE.md files reference these docs. Claude reads them on-demand when working on related features.

## Migration Notes

These documents were migrated from `littletalks-mobile/docs/guides/` on 2025-12-17. Pointer files remain in the original locations for backward compatibility.
