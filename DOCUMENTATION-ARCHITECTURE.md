# Documentation Architecture Guide

> **Meta-documentation**: This document explains how the LittleTalks documentation system works and how to maintain it.

## The Three-Tier System

```
┌─────────────────────────────────────────────────────────────────┐
│                        CONFLUENCE                                │
│  Business docs, marketing briefs, strategy                       │
│  Audience: Business partners, marketing firm, non-technical      │
│  Claude access: WebFetch when explicitly needed                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    littletalks-docs (this repo)                  │
│  Private shared technical documentation                          │
│  Audience: Claude, developers                                    │
│                                                                  │
│  ├── CLAUDE.md              ← "Mother CLAUDE" - shared context   │
│  ├── DOCUMENTATION-ARCHITECTURE.md  ← This file                  │
│  ├── shared/                ← Cross-project technical docs       │
│  └── littlepipes/           ← CI/CD platform docs (private)      │
└─────────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ Project CLAUDE.md│  │ Project CLAUDE.md│  │ Project CLAUDE.md│
│                 │  │                 │  │                 │
│ Tech stack,     │  │ Tech stack,     │  │ Tech stack,     │
│ file structure, │  │ file structure, │  │ file structure, │
│ dev commands    │  │ dev commands    │  │ dev commands    │
└─────────────────┘  └─────────────────┘  └─────────────────┘
   littletalks-         littletalks-         littletalks-
      mobile               admin                 api
```

## How It Works

### Mother CLAUDE (`littletalks-docs/CLAUDE.md`)
- **Loaded**: At session start (small, ~60 lines)
- **Contains**: Jira setup, paths, commit guidelines, shared context
- **Purpose**: Every project references this for common standards

### Project CLAUDE.md Files
- **Loaded**: At session start for whichever project you're in
- **Contains**: Project-specific tech stack, file structure, dev commands
- **First line**: References Mother CLAUDE for shared context
- **Goal**: Keep LEAN (<100 lines) to minimize context cost

### Shared Technical Docs (`littletalks-docs/shared/`)
- **Loaded**: On-demand when working on that topic
- **Contains**: Deep technical guides (journey system, analytics, RevenueCat)
- **Purpose**: Single source of truth for cross-project knowledge

### LittlePipes Docs (`littletalks-docs/littlepipes/`)
- **Loaded**: On-demand when working on CI/CD
- **Contains**: Build system architecture, costs, workflows
- **Why here**: lp_test repo is PUBLIC; these docs are private

## When to Put Docs Where

| Document Type | Location | Example |
|--------------|----------|---------|
| Business strategy | Confluence | Revenue model, marketing plans |
| Shared technical | `littletalks-docs/shared/` | Journey system, analytics events |
| CI/CD platform | `littletalks-docs/littlepipes/` | Build workflows, version management |
| Project-specific patterns | Project's `docs/guides/` | Component guides, testing patterns |
| Session continuity | Project's `docs/session_handoffs/` | Work history, blockers |
| Claude instructions | Project's `CLAUDE.md` | Tech stack, dev commands |

## Adding New Documentation

### New Shared Doc (cross-project)
1. Create in `littletalks-docs/shared/`
2. Update `shared/README.md` with entry
3. If replacing existing doc in a project, leave pointer file

### New Project-Specific Doc
1. Create in that project's `docs/guides/`
2. Update project's CLAUDE.md if Claude needs to know about it

### New CI/CD Doc
1. Create in `littletalks-docs/littlepipes/`
2. Do NOT put in lp_test (public repo)

## Maintenance Rules

### Keep Project CLAUDE.md Lean
- Target: <100 lines
- If adding content, ask: "Does Claude need this at session start?"
- If no → put in `docs/` and reference on-demand

### Single Source of Truth
- Never duplicate docs across repos
- Use pointer files: "Document moved to littletalks-docs/shared/..."
- When updating shared docs, update in ONE place

### Moving Docs to Shared
1. Copy file to `littletalks-docs/shared/` (or `littlepipes/`)
2. Replace original with pointer file
3. Update `shared/README.md`
4. Commit both repos

### Session Handoffs
- Template: `littletalks-docs/shared/session-handoff-template.md`
- Location: Each project's `docs/session_handoffs/`
- Filename: `YYYYMMDD-HHMM-brief-description.md`

## Reference Projects

| Project | Jira | Type | Notes |
|---------|------|------|-------|
| littletalks-mobile | LTD | Private | React Native app |
| littletalks-admin | LTD | Private | PHP admin dashboard |
| littletalks-api | LTD | Private | Node.js backend |
| lp_test | LP | **PUBLIC** | CI/CD workflows only - no sensitive docs! |
| WXING | CD | Personal | Reference patterns (JiraService, notifications) |

## For Future Claudes

When starting a new session:
1. Check which project you're in (working directory)
2. Read that project's CLAUDE.md (references Mother CLAUDE)
3. If working on shared features (journey, analytics, subscriptions), read relevant `littletalks-docs/shared/` doc
4. Check `docs/session_handoffs/` for recent context
5. Use TodoWrite to track your work

When creating documentation:
1. Ask: "Is this used by multiple projects?" → shared/
2. Ask: "Is this CI/CD related?" → littlepipes/
3. Ask: "Is this project-specific?" → project's docs/guides/
4. Ask: "Is this business/non-technical?" → Confluence
