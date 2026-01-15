# Checkpoint Checklist

> **The Instant Retrospective**: Ask these questions at every PR, every commit, every natural stopping point. Don't wait for problems to accumulate.

## The Meta Question

Before anything else, ask:

> **"If I had to hand this codebase to a new developer tomorrow, would they understand it without me explaining anything?"**

If the answer is "no," stop and fix it before moving forward.

---

## Pre-Flight Checklist (Before Building)

Run this BEFORE writing a new screen, component, or feature:

- [ ] **UI Primitives**: What components does this need? (Button, Input, Card, Modal, etc.)
  - Do styled versions exist in `src/components/ui/`?
  - If not: **Create them FIRST**, then build the screen
- [ ] **Translation Keys**: What user-facing strings are needed?
  - Are they in the localization files?
  - If not: **Add them FIRST**
- [ ] **Similar Screen**: Is there an existing screen to reference for patterns?
- [ ] **Data Requirements**: What data/state does this need? Do the hooks/services exist?

The goal: Never build a screen that requires primitives you don't have. Build bottom-up, not top-down.

---

## Red Flags (Stop Immediately)

If you see yourself typing any of these, **STOP** and fix it:

| Red Flag | What To Do Instead |
|----------|-------------------|
| `style={{` in a screen file | Create/use a styled component in `ui/` |
| `#RRGGBB` or `#RGB` anywhere | Use theme token (`$primary`, `$color`, `$background`) |
| Quoted English string in JSX | Use `t('_NAMESPACE.KEY')` from localization |
| `any` type | Define a proper interface/type |
| Copy-pasting styles from another file | Extract to a shared component |
| `// TODO` without a ticket | Create the ticket or fix it now |

These are the patterns that create invisible technical debt. Catch them in the moment.

---

## Quick Validation Commands

Run these to catch violations automatically. These examples are for React/TypeScript projects — adapt the patterns for your stack.

```bash
# Find inline styles in screens (should be zero)
grep -r "style={{" src/screens/

# Find hardcoded colors (should be zero outside theme/)
grep -rE "#[0-9A-Fa-f]{3,6}" src/screens/ src/components/

# Find hardcoded strings (review each - should use t())
grep -rE '>[A-Z][a-z]+.*</' src/screens/

# Find any types (should be zero)
grep -r ": any" src/
```

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

### Native SDK Integration (React Native)

When adding a native SDK (Firebase, Bugsnag, Facebook, AdMob, etc.):

- [ ] **Native initialization**: Added `SdkName.start()` to MainApplication.kt (Android) AND AppDelegate.swift (iOS)?
- [ ] **Native config**: API keys in native config files (AndroidManifest.xml, Info.plist, strings.xml), not hardcoded in JS?
- [ ] **Jest mock**: Added mock to `jest.setup.js` so tests run without native module?
- [ ] **iOS pods**: Noted in PR that `cd ios && pod install` is required after pulling?
- [ ] **Device tested**: Verified on actual device/emulator, not just unit tests passing?

> **Why this matters**: JS-side `sdk.start()` often isn't enough. Many native SDKs require initialization in the native Application/AppDelegate class BEFORE the React Native bridge loads. Missing this causes runtime crashes that unit tests won't catch.

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

## Build Verification (Before Every Commit)

**CRITICAL**: Do NOT commit until you've verified the app actually runs.

### Android
```bash
# 1. Build
cd android && ./gradlew assembleLittletalksDebug

# 2. Install
adb install -r app/build/outputs/apk/littletalks/debug/app-littletalks-debug.apk

# 3. Launch and verify
adb shell am start -n com.littletalks.app/com.littlewalksmobile.MainActivity
# Confirm: App displays correctly (not white screen)
```

### iOS
```bash
# 1. Install pods (if dependencies changed)
cd ios && pod install

# 2. Build and run
npx react-native run-ios --scheme LittleTalks
# Or via Xcode: Open .xcworkspace, select scheme, Run
```

**In commit message**, include verification status:
- `Verified: build/install/launch OK` - All steps passed
- `Verified: build OK, launch UNTESTED` - Built but couldn't test (explain why)
- `WIP: not verified` - Work in progress, don't merge

**If white screen after build**:
1. First try: `adb shell am force-stop <package> && adb shell am start ...`
2. If still broken: Restart emulator fresh
3. If still broken: Check Metro for JS errors

---

## When to Run This Checklist

| Trigger | Depth |
|---------|-------|
| Every commit | Quick mental scan + build verification |
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

### Changelog

| Date | Change |
|------|--------|
| 2026-01-15 | Added Native SDK Integration checklist after Bugsnag required native MainApplication.kt init that unit tests didn't catch; added iOS build verification commands |
| 2026-01-14 | Added Build Verification section after white screen debugging session revealed need for explicit build/install/launch verification before commits |
| 2026-01-13 | Added Pre-Flight Checklist, Red Flags, and Quick Validation Commands after instant retro caught inline styles and hardcoded strings that slipped through |
| 2026-01-09 | Initial version created |

*"The best time to catch a problem is before it becomes a pattern."*
