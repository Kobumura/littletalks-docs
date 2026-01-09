# Instant Retrospectives: Building Quality Into the Rhythm of Development

> **TL;DR**: Instead of retrospectives at the end of sprints (when problems have compounded), we run quality checkpoints at every PR and commit. One meta question drives everything: "If I had to hand this codebase to a new developer tomorrow, would they understand it without me explaining anything?"

*Who this is for: Any engineering team tired of technical debt accumulating faster than they can pay it down. Works with any language, framework, or AI tool.*

---

## The Problem

Most teams do retrospectives:
- **Weekly or bi-weekly** — Problems compound for days before anyone asks "should we have done this differently?"
- **After incidents** — Reactive, not preventive. The damage is done.
- **At project end** — Too late to fix architecture. You're just documenting regrets.

The result: Technical debt accumulates in the gaps between retrospectives. By the time anyone notices, patterns have spread, workarounds have become load-bearing, and "we'll fix it later" has become permanent.

**Our wake-up call**: A React Native app that had passed through multiple development teams over several years. Each team added features without asking hard questions. The codebase became so fragile that upgrading React Native versions became impossible—we tried for weeks, failed, and eventually decided a greenfield rebuild was faster than fixing the mess.

We didn't want to repeat that with the new codebase.

---

## The Solution: Checkpoint Questions at Every Stop

Instead of waiting for scheduled retrospectives, we ask quality questions at every natural stopping point:

| Trigger | Depth |
|---------|-------|
| Every commit | Quick mental scan |
| Every PR | Full checklist review |
| End of work session | Retrospective + session handoff |
| Before release | Full checklist + QA |
| After bug fix | "How did this slip through?" analysis |

The key insight: **The best time to catch a problem is before it becomes a pattern.**

---

## The Meta Question

Before diving into specifics, we start with one question that cuts through everything:

> **"If I had to hand this codebase to a new developer tomorrow, would they understand it without me explaining anything?"**

If the answer is "no," stop and fix it before moving forward.

This single question catches:
- Unclear naming
- Missing documentation
- Clever-but-confusing code
- Implicit assumptions
- Tribal knowledge that isn't written down

When working with AI collaborators like Claude, this question becomes even more important—every new session is essentially a "new developer" with no memory of previous context.

---

## The Checkpoint Checklist

Here's what we review at every PR and significant commit:

### Architecture & Design
- [ ] **Single Responsibility**: Does each file/function do ONE thing well?
- [ ] **Separation of Concerns**: Is business logic separate from UI? Data fetching separate from display?
- [ ] **DRY (Don't Repeat Yourself)**: Are we repeating code that should be abstracted?
- [ ] **YAGNI (You Ain't Gonna Need It)**: Are we over-engineering for hypothetical future needs?
- [ ] **Configuration over Hardcoding**: Is this hardcoded when it should come from config/database?

### Code Quality
- [ ] **No Magic Numbers/Strings**: Are constants named and defined somewhere sensible?
- [ ] **No Inline Styles** (frontend): Using the theme/design system?
- [ ] **Localization Ready**: No hardcoded user-facing strings?
- [ ] **Type Safety**: No `any` types sneaking in? Strict mode enabled?
- [ ] **Naming Clarity**: Would someone understand what `handleClick` or `processData` does without reading the implementation?

### Error Handling & Edge Cases
- [ ] **Graceful Failures**: What happens when this fails? Does the user see a helpful message or a crash?
- [ ] **Null/Empty Handling**: What if the data is null? Empty array? Undefined?
- [ ] **Network Failures**: What if the API is down? Timeout? 500 error?
- [ ] **Loading States**: Does the user know something is happening?
- [ ] **Boundary Conditions**: First item? Last item? Zero items? Max items?

### Testing & Reliability
- [ ] **Testable**: Is this structured so it CAN be tested?
- [ ] **Tested**: Did we actually write tests?
- [ ] **Test Coverage**: Are the important paths covered, not just the happy path?
- [ ] **Regression Prevention**: If this broke before, is there a test to prevent it breaking again?

### Performance & Security
- [ ] **No Secrets in Code**: All keys/tokens in config files or environment variables?
- [ ] **No Blocking Operations**: Long operations are async?
- [ ] **Memory Leaks**: Cleaning up subscriptions, listeners, timers?
- [ ] **N+1 Queries**: Database calls in loops?
- [ ] **Bundle Size** (frontend): Adding a huge dependency for a small feature?

### Maintainability & Documentation
- [ ] **Self-Documenting**: Is the code clear enough that comments aren't needed?
- [ ] **Necessary Comments**: Complex logic explained? "Why" not just "what"?
- [ ] **Consistent Patterns**: Does this follow the patterns established elsewhere in the codebase?
- [ ] **Updated Docs**: If this changes behavior, are docs/READMEs updated?

---

## Project-Specific Questions

The generic checklist is a starting point. Add questions specific to your project type:

### White-Label / Multi-Tenant Projects
- [ ] **Multi-Brand Ready**: Will this work for Brand #2, Brand #3?
- [ ] **Platform Agnostic**: Works on iOS AND Android?
- [ ] **Config-Driven**: Is brand-specific behavior coming from config, not code?

### API Projects
- [ ] **Backward Compatible**: Will existing clients break?
- [ ] **Documented Endpoint**: Is the API contract documented?
- [ ] **Rate Limited**: Should this endpoint have rate limiting?

### Admin/Dashboard Projects
- [ ] **Permission Checked**: Is this action properly authorized?
- [ ] **Audit Logged**: Should this action be logged for audit purposes?

### Database Operations
- [ ] **Migration Reversible**: Can this migration be rolled back?
- [ ] **Index Considered**: Will this query benefit from an index?
- [ ] **Data Backfill**: Does existing data need updating?

---

## The Accountability Question

At the end of every work session, we ask one more question:

> **"What did we build today that we might regret in 6 months?"**

If something comes to mind, either fix it now or create a ticket to address it. Don't let it become invisible.

This question has saved us multiple times. It surfaces the "I know this is a little hacky but..." moments that otherwise get buried under the pressure to ship.

---

## Why This Works

### 1. Problems Are Caught in Hours, Not Days
A weekly retrospective means a bad pattern can spread for 5 days before anyone questions it. Checkpoint questions catch it on the first commit.

### 2. Context Is Fresh
When you ask "why did we do it this way?" at the end of a sprint, you're relying on memory. When you ask during the PR, the reasoning is still in your head.

### 3. It's Lightweight
This isn't a meeting. It's a mental checklist that takes 30 seconds for small commits and 5 minutes for significant PRs. The overhead is minimal compared to the cost of technical debt.

### 4. It Scales with AI Collaboration
When working with AI tools, every session starts fresh. The checkpoint questions help ensure that what gets built in one session doesn't create confusion in the next.

### 5. It's Preventive, Not Reactive
Traditional retrospectives ask "what went wrong?" Instant retrospectives ask "what could go wrong if we don't address this now?"

---

## Real Example: The Greenfield Rebuild

We used this approach while rebuilding a React Native app from scratch. The old codebase had:
- Inline styles scattered across every component
- State management that nobody fully understood
- "Temporary" workarounds that had been there for years
- Tests that existed but didn't catch real bugs

For the new codebase, we committed to checkpoint questions from day one:

**Day 1**: Set up strict TypeScript, ESLint with `no-inline-styles: error`, and path aliases. The linter would enforce some checklist items automatically.

**Every PR**: Asked the meta question. If the answer was "a new developer would be confused by this," we refactored before merging.

**Every session end**: Created a session handoff document capturing what was built and any concerns.

**Result**: After several weeks of development, the codebase remained navigable. A fresh Claude session could understand the architecture without lengthy explanations. The patterns established early continued to hold.

---

## Common Objections

### "This will slow us down"
The checklist takes 30 seconds to scan mentally for small changes. The time saved by not accumulating technical debt far exceeds this cost. Ask anyone who's spent a week trying to upgrade a framework in a messy codebase.

### "We already do code review"
Code review typically focuses on "does this work?" and "is there an obvious bug?" The checkpoint questions focus on "will we regret this?" and "can someone else understand this?" They're complementary.

### "Some of these don't apply to my project"
Remove what doesn't apply. Add what does. The checklist is a starting point, not a mandate. The meta question is the only universal requirement.

### "My team won't adopt this"
Start alone. If it makes your code better, others will notice. If it doesn't, stop doing it. The approach proves itself through results, not mandates.

---

## How to Start

1. **Copy the checklist** into your project's documentation
2. **Customize it** for your project type and tech stack
3. **Ask the meta question** at your next PR: "Would a new developer understand this?"
4. **Add the accountability question** to your end-of-day routine
5. **Iterate** based on what patterns you catch

You don't need buy-in from your whole team. You don't need a new tool. You just need to ask better questions at natural stopping points.

---

## The Core Insight

**Retrospectives are too late. Quality is built in the rhythm of development, not bolted on at the end.**

The best time to ask "should we do this differently?" is while you're still doing it.

---

*This approach emerged from a greenfield rebuild where we wanted to prevent the technical debt accumulation that had made the previous codebase unmaintainable. It's now part of our [Mother CLAUDE documentation system](./claude-documentation-system.md) and gets referenced at every checkpoint across all projects.*

*The checklist is open. The approach is portable. The goal is simple: catch problems before they become patterns.*
