# Checkpoint Checklist

> **The Instant Retrospective**: Ask these questions at every PR, every commit, every natural stopping point. Don't wait for problems to accumulate.

## The Meta Question

Before anything else, ask:

> **"If I had to hand this codebase to a new developer tomorrow, would they understand it without me explaining anything?"**

If the answer is "no," stop and fix it before moving forward.

---

## Architecture & Design

- [ ] **Single Responsibility**: Does each file/function do ONE thing well?
- [ ] **Separation of Concerns**: Is business logic separate from UI? Data fetching separate from display?
- [ ] **DRY (Don't Repeat Yourself)**: Are we repeating code that should be abstracted?
- [ ] **YAGNI (You Ain't Gonna Need It)**: Are we over-engineering for hypothetical future needs?
- [ ] **Configuration over Hardcoding**: Is this hardcoded when it should come from config/database?

## Code Quality

- [ ] **No Magic Numbers/Strings**: Are constants named and defined somewhere sensible?
- [ ] **No Inline Styles** (frontend): Using the theme/design system?
- [ ] **Localization Ready**: No hardcoded user-facing strings?
- [ ] **Type Safety**: No `any` types sneaking in? Strict mode enabled?
- [ ] **Naming Clarity**: Would someone understand what `handleClick` or `processData` does without reading the implementation?

## Error Handling & Edge Cases

- [ ] **Graceful Failures**: What happens when this fails? Does the user see a helpful message or a crash?
- [ ] **Null/Empty Handling**: What if the data is null? Empty array? Undefined?
- [ ] **Network Failures**: What if the API is down? Timeout? 500 error?
- [ ] **Loading States**: Does the user know something is happening?
- [ ] **Boundary Conditions**: First item? Last item? Zero items? Max items?

## Testing & Reliability

- [ ] **Testable**: Is this structured so it CAN be tested?
- [ ] **Tested**: Did we actually write tests?
- [ ] **Test Coverage**: Are the important paths covered, not just the happy path?
- [ ] **Regression Prevention**: If this broke before, is there a test to prevent it breaking again?

## Performance & Security

- [ ] **No Secrets in Code**: All keys/tokens in config files or environment variables?
- [ ] **No Blocking Operations**: Long operations are async?
- [ ] **Memory Leaks**: Cleaning up subscriptions, listeners, timers?
- [ ] **N+1 Queries**: Database calls in loops?
- [ ] **Bundle Size** (frontend): Adding a huge dependency for a small feature?

## Maintainability & Documentation

- [ ] **Self-Documenting**: Is the code clear enough that comments aren't needed?
- [ ] **Necessary Comments**: Complex logic explained? "Why" not just "what"?
- [ ] **Consistent Patterns**: Does this follow the patterns established elsewhere in the codebase?
- [ ] **Updated Docs**: If this changes behavior, are docs/READMEs updated?

## Project-Specific Questions

Add project-specific questions here. Examples:

### White-Label Projects (LittleWalks)
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

---

## When to Run This Checklist

| Trigger | Depth |
|---------|-------|
| Every commit | Quick mental scan |
| Every PR | Full checklist review |
| End of work session | Retrospective + session handoff |
| Before release | Full checklist + QA |
| After bug fix | "How did this slip through?" analysis |

## The Accountability Question

At the end of every session, ask:

> **"What did we build today that we might regret in 6 months?"**

If something comes to mind, either fix it now or create a ticket to address it.

---

## History

This checklist emerged from the LittleWalks greenfield rebuild (January 2026) as a way to prevent the technical debt accumulation that plagued the original LittleTalks codebase. It's designed to be:

- **Lightweight**: Quick to scan, not a burden
- **Universal**: Applies to any language/framework
- **Living**: Add project-specific questions as needed

*"The best time to catch a problem is before it becomes a pattern."*
