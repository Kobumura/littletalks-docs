# How We Built a Documentation System That Makes Claude Productive Immediately

> **TL;DR**: We created a three-tier documentation architecture that lets new Claude sessions be productive within minutes, not hours. First real-world test: 8/10 rating from Claude on first try.

---

## The Problem

Working with Claude Code across multiple projects, we kept hitting the same issues:

1. **Context switching pain** - Starting a new session meant re-explaining project structure, conventions, and history
2. **Scattered documentation** - Important info spread across READMEs, code comments, and tribal knowledge
3. **Duplication and drift** - Same information (Jira setup, commit guidelines) copy-pasted across repos, getting out of sync
4. **Legacy project onboarding** - Rewriting old codebases required extensive explanation of "why things were built this way"

**The goal**: Get Claude productive immediately, whether it's a fresh session or a completely new project.

---

## The Solution: Three-Tier Documentation Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     CONFLUENCE                          │
│  Business docs, marketing, non-technical                │
│  → Claude fetches on-demand via WebFetch                │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│              Shared Docs Repo (Mother CLAUDE)           │
│  Cross-project standards, deep technical guides         │
│  → Loaded at session start (lean) + on-demand (deep)    │
└─────────────────────────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Project CLAUDE.md│ │ Project CLAUDE.md│ │ Project CLAUDE.md│
│ Tech stack,      │ │ Tech stack,      │ │ Tech stack,      │
│ file structure,  │ │ file structure,  │ │ file structure,  │
│ dev commands     │ │ dev commands     │ │ dev commands     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Tier 1: Mother CLAUDE (Shared Standards)

A dedicated repo (`littletalks-docs`) containing:
- **CLAUDE.md** (~70 lines) - Jira setup, screenshot locations, commit guidelines, project paths
- **shared/** - Deep technical docs (journey system, analytics, API contracts)
- **DOCUMENTATION-ARCHITECTURE.md** - Meta-documentation explaining the system itself

Every project's CLAUDE.md starts with:
```markdown
> **Shared standards**: See `littletalks-docs/CLAUDE.md` for Jira, screenshots, commit guidelines.
```

### Tier 2: Project CLAUDE.md (Lean & Specific)

Each project has its own CLAUDE.md that's:
- **Lean** (<100 lines) - Minimizes context cost
- **Specific** - Tech stack, file structure, dev commands for THIS project
- **Referenced** - Points to Mother CLAUDE for shared standards
- **Actionable** - Claude can start working immediately

### Tier 3: On-Demand Deep Docs

Detailed guides live in `shared/` and are read only when needed:
- Journey system architecture (1,251 lines)
- Analytics event tracking
- RevenueCat integration
- API contracts

This keeps session context lean while making deep knowledge accessible.

---

## Key Design Decisions

### 1. Single Source of Truth
When we moved docs to the shared repo, we left **pointer files** in the original locations:
```markdown
# Document Moved
This document has been moved to `littletalks-docs/shared/journey-system.md`
```

No duplication = no drift.

### 2. Security Built In
PHP projects deployed to web servers get `.htaccess` in their `docs/` folder:
```apache
Require all denied
```

This is documented in Mother CLAUDE so new projects know to add it.

### 3. Session Handoffs
A standardized template (`session-handoff-template.md`) ensures continuity:
- What was accomplished
- Current state
- Lessons learned
- Next steps
- Key files modified

Each project has a `docs/session_handoffs/` directory for these.

### 4. Self-Documenting
The `DOCUMENTATION-ARCHITECTURE.md` file explains the entire system. New Claudes can understand the architecture without human explanation.

---

## The Legacy Project Challenge

Our biggest test: preparing a 2011-2020 football pool codebase for a greenfield rewrite.

### What We Prepared

**football-pool-legacy/** (reference archive):
- `CLAUDE.md` - Business logic locations, DO NOT MODIFY warning
- `EVOLUTION.md` - Historical context template with sections for:
  - Data volumes
  - Feature history & lessons learned
  - User/admin pain points
  - Known bugs & workarounds
  - Old UI screenshots

**football/** (greenfield):
- `CLAUDE.md` - Full requirements, phases, success criteria
- `docs/session_handoffs/` - Ready for continuity
- Pre-researched docs (ESPN score feeds)

### The Result

First Claude session on the football project:

> **Feedback on the Prep System**: 8/10
>
> The setup was genuinely helpful. CLAUDE.md gave immediate context. Legacy repo access was critical. The empty EVOLUTION.md and lack of "lessons learned" notes were the main gaps.

**Productive immediately** - Claude went into planning mode after just a few questions.

---

## The Legacy Project Prep Checklist

Based on feedback, we created a checklist for preparing legacy codebases:

### Required
- [ ] CLAUDE.md with Mother CLAUDE reference
- [ ] Path to legacy repo
- [ ] Jira project code
- [ ] Tech stack (old and new)

### Highly Recommended
- [ ] EVOLUTION.md with:
  - Data volumes (helps migration planning)
  - Feature history (why things were built)
  - User pain points (prioritization)
  - Admin pain points (what to fix)
  - Known bugs/workarounds
  - Old UI screenshots

### Pre-Research
- [ ] External API documentation
- [ ] Integration requirements
- [ ] SQL dumps (empty schema + full data)

---

## Lessons Learned

### What Worked
1. **Lean CLAUDE.md files** - <100 lines keeps context cost low
2. **Single source of truth** - No duplication across repos
3. **On-demand deep docs** - Don't load everything at session start
4. **Legacy repo access** - Claude examining actual code beats any description
5. **Self-documenting architecture** - The system explains itself

### What Could Be Better
1. **EVOLUTION.md needs content** - Empty templates don't help
2. **Historical context matters** - "Why was it built this way?" is valuable
3. **Data volumes help** - Claude needs to think about scale
4. **Screenshots calibrate expectations** - Even rough ones help

### The Meta Insight
**Documentation is a product**. It needs:
- User research (what does Claude need?)
- Iteration (improve based on feedback)
- Maintenance (keep it current)
- Self-documentation (explain how it works)

---

## Try It Yourself

1. **Create a shared docs repo** with Mother CLAUDE
2. **Keep project CLAUDE.md lean** (<100 lines)
3. **Move shared docs** to single source of truth
4. **Add security** (.htaccess for web-deployed PHP)
5. **Create EVOLUTION.md** for legacy projects
6. **Get feedback** from Claude on first session
7. **Iterate** based on what's missing

---

## Appendix: Our Project Ecosystem

```
Kobumura (Company Projects):
├── littletalks-mobile      # React Native app
├── littletalks-admin       # PHP admin dashboard
├── littletalks-api         # Node.js backend
├── lp_test                 # CI/CD (PUBLIC repo)
└── littletalks-docs        # Shared docs (Mother CLAUDE)

Personal (Use shared standards):
├── WXING                   # Reference patterns
├── football                # Greenfield rewrite
└── football-pool-legacy    # Archive (2011-2020)
```

All projects reference the same Mother CLAUDE for consistency, while maintaining project-specific context in their own CLAUDE.md files.

---

*This article documents a system built collaboratively between Dorothy and Claude Code over the course of a single session. The 8/10 rating from a fresh Claude session validated the approach - there's always room for improvement, but "productive immediately" was achieved.*
