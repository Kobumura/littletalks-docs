# Automated Development Workflow Plan

**Epic**: LP-XX (Jira-Integrated CI/CD Automation)
**Status**: Planning
**Last Updated**: December 2, 2025

---

## Executive Summary

This document outlines an automated workflow system that connects Jira ticket management, GitHub branches, LittlePipes CI/CD, and app store testing tracks. The goal is to create a seamless flow from development to tester approval to production release with minimal manual intervention.

---

## Current State

### What We Have
- **LittlePipes**: Operational CI/CD with Fastlane, public/private repo dispatch pattern
- **Jira Smart Commits**: Linking commits to tickets via ticket IDs in commit messages
- **Manual Trigger**: LittlePipes builds triggered manually via GitHub Actions UI
- **Basic Statuses**: LTD has "To Do", "In Progress", "Done"; LP has "Backlog", "Selected for Development", "In Progress", "Done"

### What We Need
- Automated build triggers based on commit keywords
- Jira status updates from GitHub Actions
- Tester approval workflow via Jira
- Automated PR creation and merging
- Conflict detection and notification
- LittlePipes trigger on successful merge to `dev`

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              DEVELOPER WORKFLOW                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. Create feature branch from dev                                          │
│     └── fix/LTD-232-chat-input-text-color                                  │
│                                                                              │
│  2. Develop & test locally                                                  │
│                                                                              │
│  3. Commit with [RFB] when ready for tester review                         │
│     └── "fix(ui): chat input text [LTD-232] [RFB]"                         │
│                                                                              │
│  4. Push to GitHub                                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          GITHUB ACTION: RFB DETECTOR                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Triggers on: push to any branch with [RFB] in commit message               │
│                                                                              │
│  Steps:                                                                      │
│  1. Parse commit message for ticket ID (LTD-XXX)                            │
│  2. Trigger LittlePipes internal test build                                 │
│  3. Wait for build completion                                                │
│  4. Update Jira ticket:                                                      │
│     - Status → "Ready for Testing"                                          │
│     - Add comment with build link (TestFlight/Internal Track)               │
│     - Assign to designated tester (Stephen)                                 │
│  5. Post Slack notification (optional)                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              TESTER WORKFLOW                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Stephen receives Jira notification:                                        │
│  - Ticket assigned to him                                                   │
│  - Status is "Ready for Testing"                                            │
│  - Comment contains build install link                                      │
│                                                                              │
│  Testing:                                                                    │
│  1. Install build from TestFlight/Internal Track                            │
│  2. Test the feature described in ticket                                    │
│  3. Update Jira status:                                                      │
│     - "Approved" → Feature is good, proceed to merge                        │
│     - "Needs Work" → Problems found, back to developer                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
           [Needs Work]                       [Approved]
                    │                               │
                    ▼                               ▼
┌──────────────────────────────┐  ┌──────────────────────────────────────────┐
│    BACK TO DEVELOPER         │  │     GITHUB ACTION: APPROVAL HANDLER      │
├──────────────────────────────┤  ├──────────────────────────────────────────┤
│                              │  │                                          │
│  - Ticket reassigned to DJ   │  │  Triggers on: Jira webhook (status →    │
│  - Developer makes fixes     │  │               "Approved")                │
│  - Commit with [RFB] again   │  │                                          │
│  - Cycle repeats             │  │  Steps:                                  │
│                              │  │  1. Parse ticket for branch name         │
└──────────────────────────────┘  │  2. Check for conflicts with dev         │
                                  │     - IF CONFLICTS:                      │
                                  │       - Notify developer + tester        │
                                  │       - Update ticket: "Blocked"         │
                                  │       - STOP                             │
                                  │     - IF CLEAN:                          │
                                  │       - Create PR to dev                 │
                                  │       - Auto-merge PR                    │
                                  │       - Update ticket: "Merged"          │
                                  │                                          │
                                  └──────────────────────────────────────────┘
                                                    │
                                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      GITHUB ACTION: DEV MERGE HANDLER                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Triggers on: push to dev branch                                            │
│                                                                              │
│  Steps:                                                                      │
│  1. Trigger LittlePipes production build                                    │
│  2. Deploy to Closed Testing track (Google) / TestFlight (iOS)              │
│  3. Update all merged tickets: "Released to Testing"                        │
│  4. Post Slack summary of what's in the build                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Trigger Keywords

| Keyword | Location | Action |
|---------|----------|--------|
| `[RFB]` | Commit message | **R**eady **F**or **B**uild - triggers internal test build |
| `[BATCH:name]` | Commit message | Groups builds together (future enhancement) |
| `[SKIP-CI]` | Commit message | Skip all CI (existing convention) |

### Example Commit Messages

```bash
# Trigger internal build for tester
git commit -m "fix(ui): chat input text color [LTD-232] [RFB]"

# Normal commit, no build trigger
git commit -m "fix(ui): chat input text color [LTD-232]"

# Batch multiple features (future)
git commit -m "feat: new dashboard [LTD-240] [RFB] [BATCH:december-release]"
```

---

## Jira Status Workflow

### New Statuses Added to LTD Project

| Status | ID | Category | Description | Triggered By |
|--------|-----|----------|-------------|--------------|
| **Ready for Testing** | 10111 | In Progress (yellow) | Build ready, awaiting tester | GitHub Action (on [RFB] build complete) |
| **Approved** | 10112 | In Progress (yellow) | Tester approved, ready to merge | Tester (Stephen) |

### Simplified Status Flow

```
To Do → In Progress → Ready for Testing → Approved → Done
              ↑              │                 │
              │              │                 │
              └──────────────┴─────────────────┘
                    (if issues found, back to In Progress)
```

**Design Decision:** We intentionally kept this simple with only 2 new statuses:
- If tester finds issues → move back to "In Progress" (developer fixes and re-commits with [RFB])
- If merge conflicts → move back to "In Progress" with a comment explaining the conflict
- After merge to dev → move directly to "Done" (no intermediate "Merged" or "Released" status)

This keeps the workflow simple for Stephen while still enabling full automation.

---

## Technical Implementation

### Component 1: RFB Build Trigger (GitHub Action)

**File**: `.github/workflows/rfb-build-trigger.yml`
**Location**: `littletalks-mobile` (private repo)

```yaml
name: RFB Build Trigger

on:
  push:
    branches-ignore:
      - main
      - dev

jobs:
  check-rfb:
    runs-on: ubuntu-latest
    outputs:
      has_rfb: ${{ steps.check.outputs.has_rfb }}
      ticket_id: ${{ steps.check.outputs.ticket_id }}
      branch_name: ${{ steps.check.outputs.branch_name }}
    steps:
      - name: Check for [RFB] in commit message
        id: check
        run: |
          COMMIT_MSG="${{ github.event.head_commit.message }}"
          if [[ "$COMMIT_MSG" == *"[RFB]"* ]]; then
            echo "has_rfb=true" >> $GITHUB_OUTPUT
            # Extract ticket ID (LTD-XXX)
            TICKET=$(echo "$COMMIT_MSG" | grep -oE 'LTD-[0-9]+' | head -1)
            echo "ticket_id=$TICKET" >> $GITHUB_OUTPUT
            echo "branch_name=${{ github.ref_name }}" >> $GITHUB_OUTPUT
          else
            echo "has_rfb=false" >> $GITHUB_OUTPUT
          fi

  trigger-build:
    needs: check-rfb
    if: needs.check-rfb.outputs.has_rfb == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Trigger LittlePipes Internal Build
        run: |
          curl -X POST \
            -H "Authorization: token ${{ secrets.LITTLEPIPES_DISPATCH_TOKEN }}" \
            -H "Accept: application/vnd.github.v3+json" \
            https://api.github.com/repos/Kobumura/littlepipes/dispatches \
            -d '{
              "event_type": "internal-build",
              "client_payload": {
                "branch": "${{ needs.check-rfb.outputs.branch_name }}",
                "ticket_id": "${{ needs.check-rfb.outputs.ticket_id }}",
                "build_type": "internal"
              }
            }'

      - name: Update Jira - Ready for Testing
        run: |
          # Transition ticket to "Ready for Testing"
          # Add comment with build link
          # Assign to Stephen
          # (Implementation details in LP-XX subtask)
```

### Component 2: Jira Webhook Receiver (GitHub Action)

**File**: `.github/workflows/jira-approval-handler.yml`
**Location**: `littlepipes` (public repo) or serverless function

```yaml
name: Jira Approval Handler

on:
  repository_dispatch:
    types: [jira-status-change]

jobs:
  handle-approval:
    if: github.event.client_payload.status == 'Approved'
    runs-on: ubuntu-latest
    steps:
      - name: Check for conflicts
        run: |
          # Fetch both branches
          # Check merge compatibility
          # If conflicts, update Jira and notify

      - name: Create and merge PR
        if: success()
        run: |
          # Create PR from feature branch to dev
          # Auto-merge
          # Update Jira status to "Merged"
```

### Component 3: Dev Merge Handler (GitHub Action)

**File**: `.github/workflows/dev-merge-handler.yml`
**Location**: `littletalks-mobile`

```yaml
name: Dev Merge Handler

on:
  push:
    branches:
      - dev

jobs:
  trigger-production-build:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger LittlePipes Production Build
        run: |
          curl -X POST \
            -H "Authorization: token ${{ secrets.LITTLEPIPES_DISPATCH_TOKEN }}" \
            https://api.github.com/repos/Kobumura/littlepipes/dispatches \
            -d '{
              "event_type": "production-build",
              "client_payload": {
                "branch": "dev",
                "build_type": "closed-testing"
              }
            }'

      - name: Update merged tickets
        run: |
          # Find all tickets in this merge
          # Update status to "Released to Testing"
```

### Component 4: Jira Webhook Configuration

**Setup in Jira**:
1. Project Settings → Automation
2. Create rule: When status changes to "Approved"
3. Action: Send web request to GitHub repository dispatch endpoint

```json
{
  "event_type": "jira-status-change",
  "client_payload": {
    "ticket_id": "{{issue.key}}",
    "status": "{{issue.status.name}}",
    "branch": "{{issue.customfield_XXXXX}}"  // Custom field for branch name
  }
}
```

---

## Required Secrets

### In `littletalks-mobile` (private repo)
- `JIRA_API_TOKEN` - For updating Jira tickets
- `JIRA_EMAIL` - Jira account email
- `LITTLEPIPES_DISPATCH_TOKEN` - PAT for triggering littlepipes workflows
- `SLACK_WEBHOOK_URL` - For notifications (optional)

### In `littlepipes` (public repo)
- Already has build secrets
- Needs: `JIRA_API_TOKEN`, `JIRA_EMAIL` for status updates
- Needs: `CI_ACCESS_TOKEN` for PR operations on littletalks-mobile

---

## Epic & Ticket Breakdown

### Epic: [LP-28](https://kobumura.atlassian.net/browse/LP-28) - Jira-Integrated CI/CD Automation

**Description**: Implement automated workflow connecting Jira, GitHub, and LittlePipes for seamless development-to-testing pipeline.

### Tickets

| ID | Title | Priority | Status |
|----|-------|----------|--------|
| [LP-29](https://kobumura.atlassian.net/browse/LP-29) | Add workflow statuses to LTD project | Highest | ✅ Done |
| [LP-30](https://kobumura.atlassian.net/browse/LP-30) | RFB Build Trigger GitHub Action | Highest | To Do |
| [LP-31](https://kobumura.atlassian.net/browse/LP-31) | Jira Status Update Integration | Highest | To Do |
| [LP-32](https://kobumura.atlassian.net/browse/LP-32) | Jira Webhook Configuration for Approval | High | To Do |
| [LP-33](https://kobumura.atlassian.net/browse/LP-33) | Approval Handler - Conflict Detection and Auto-Merge | High | To Do |
| [LP-34](https://kobumura.atlassian.net/browse/LP-34) | Dev Merge Handler - Trigger LittlePipes Production Build | High | To Do |
| [LP-35](https://kobumura.atlassian.net/browse/LP-35) | Branch Name Custom Field for LTD Tickets | Medium | To Do |
| [LP-36](https://kobumura.atlassian.net/browse/LP-36) | Workflow Documentation for Developers and Testers | High | To Do |
| [LP-37](https://kobumura.atlassian.net/browse/LP-37) | Batching Support with [BATCH:name] Keyword | Low | Future |

---

## Testing Plan

### Phase 1: Manual Integration Testing
1. Create test ticket LTD-TEST
2. Create branch `fix/LTD-TEST-workflow-test`
3. Make trivial change, commit with `[RFB]`
4. Verify:
   - Build triggers
   - Jira status updates
   - Build link appears in comment

### Phase 2: Approval Flow Testing
1. Change ticket to "Approved" manually
2. Verify:
   - Conflict check runs
   - PR created
   - PR auto-merged
   - Ticket status updated

### Phase 3: End-to-End Testing
1. Full workflow from ticket creation to "Released to Testing"
2. Test conflict scenario
3. Test "Needs Work" loop

---

## Rollout Plan

### Week 1: Foundation
- Add new Jira statuses to LTD project
- Create RFB Build Trigger action
- Test basic [RFB] detection

### Week 2: Jira Integration
- Implement Jira API updates from GitHub
- Configure Jira webhook
- Test status update flow

### Week 3: Approval Automation
- Implement approval handler
- Add conflict detection
- Implement auto-merge

### Week 4: Production Integration
- Connect to LittlePipes production builds
- Add dev merge handler
- Full end-to-end testing

### Week 5: Polish & Documentation
- Add Slack notifications
- Write user documentation
- Train Stephen on new workflow

---

## Success Metrics

- **Build Trigger Time**: < 30 seconds from push to build start
- **Status Update Time**: < 10 seconds from build complete to Jira update
- **Merge Time**: < 2 minutes from approval to merged
- **Conflict Detection Accuracy**: 100%
- **Developer Time Saved**: 15+ minutes per feature (no manual PR, no manual status updates)

---

## Future Enhancements

1. **Batching**: Group multiple features into single release
2. **Rollback**: One-click revert of released features
3. **Metrics Dashboard**: Track cycle times, approval rates
4. **Auto-assignment**: AI-based tester assignment based on feature type
5. **Release Notes Generation**: Auto-generate from merged ticket descriptions

---

## Appendix A: Jira API Reference

### Transition Ticket
```bash
curl -X POST \
  -H "Authorization: Basic $(echo -n 'email:token' | base64)" \
  -H "Content-Type: application/json" \
  "https://kobumura.atlassian.net/rest/api/3/issue/LTD-232/transitions" \
  -d '{"transition": {"id": "XX"}}'
```

### Add Comment
```bash
curl -X POST \
  -H "Authorization: Basic $(echo -n 'email:token' | base64)" \
  -H "Content-Type: application/json" \
  "https://kobumura.atlassian.net/rest/api/3/issue/LTD-232/comment" \
  -d '{"body": {"type": "doc", "version": 1, "content": [...]}}'
```

### Assign User
```bash
curl -X PUT \
  -H "Authorization: Basic $(echo -n 'email:token' | base64)" \
  -H "Content-Type: application/json" \
  "https://kobumura.atlassian.net/rest/api/3/issue/LTD-232/assignee" \
  -d '{"accountId": "ACCOUNT_ID"}'
```

---

## Appendix B: Status IDs

| Status Name | Status ID | Notes |
|-------------|-----------|-------|
| To Do | 10006 | Existing |
| In Progress | 10007 | Existing |
| Ready for Testing | 10111 | **NEW** - Created 2025-12-02 |
| Approved | 10112 | **NEW** - Created 2025-12-02 |
| Done | 10008 | Existing |

**Note:** Transition IDs will be determined after statuses are added to the workflow via Jira UI.
