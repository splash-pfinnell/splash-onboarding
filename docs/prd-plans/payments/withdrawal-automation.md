---
type: prd
title: "Withdrawal Automation"
slug: withdrawal-automation
version: 1.0.0
template_version: 1.3.0
domain: payments
product_line: admin
status: draft
created: 2026-06-09
updated: 2026-06-09
author: pfinnell
source: brainstorm
target_repo: splash-app
source_one_pager: payments/withdrawal-automation-1pager.md
status_history:
  - status: draft
    date: 2026-06-09
review_passes: []
tags:
  - payments
  - withdrawals
  - automation
  - admin
---

# Withdrawal Automation

## Problem Statement

Withdrawal automation at Splash Sports runs on a single global rule — any withdrawal at or below $500 from an account with only one linked payment method is auto-processed. There is no way to adjust this threshold per payment method, and there is no in-product lever to pause automation when a risk event, compliance hold, or operational issue arises. Any change to automation behavior requires an engineering deployment. This creates unacceptable response time during incidents and prevents Finance and Compliance from tuning automation to match the risk profile of each payment method.

## User Stories

- As a **Super Admin**, I want to set a per-method dollar threshold for withdrawal automation, so that I can tune auto-processing limits to match the risk profile of each payment method without requiring an engineering deploy.
- As a **Super Admin**, I want to pause automation for a specific payment method, so that I can immediately stop new withdrawals from being auto-processed during a risk event or compliance hold without affecting withdrawals already in progress.
- As a **Trust & Safety team member**, I want to view the current automation settings for each payment method in read-only mode, so that I have visibility into the automation state without the ability to inadvertently change it.
- As a **Super Admin**, I want to see who last modified automation settings and when, so that I have an audit trail for compliance and incident review.

## Requirements

### Functional Requirements

1. The system SHALL add a **Withdrawal Automation** tab to the Payments admin page.
2. The tab SHALL display one row per supported payment method: **Aeropay, Card, PayPal, Venmo, Skrill**.
3. Each row SHALL display the payment method name, its threshold (or "Manual only" for Skrill), its pause state, and the last-modified timestamp and admin username.
4. For methods with automation enabled (Aeropay, Card, PayPal, Venmo), each row SHALL provide an editable dollar amount field for the automation threshold.
5. The system SHALL auto-process a withdrawal when ALL of the following conditions are met:
   a. The withdrawal amount is at or below the method's configured threshold.
   b. The user has exactly one payment method linked to their account.
   c. Automation for that method is not paused.
6. Withdrawals that do not meet all conditions in requirement 5 SHALL be routed to the existing manual withdrawal processing queue.
7. Each method row SHALL provide a pause/resume toggle that immediately halts automated processing of **new** withdrawal requests for that method.
8. Pausing a method SHALL NOT affect withdrawals already in a `Processing` state — those continue to completion uninterrupted.
9. A paused method SHALL display a visible status indicator (e.g., "Paused") so the state is unambiguous at a glance.
10. The Skrill row SHALL be displayed as read-only with automation permanently disabled ("Manual only") — no threshold input or pause toggle.
11. All changes to automation settings (threshold edits, pause/resume) SHALL be written to an audit log capturing: the admin username, the action taken, the previous value, the new value, and a UTC timestamp.
12. Changes SHALL take effect immediately for new withdrawal requests upon save — no deploy required.
13. Before any change is applied (threshold edit or pause/resume toggle), the system SHALL present a confirmation dialog summarizing the pending change. The change is only committed upon explicit confirmation.
14. The audit trail SHALL be viewable within the admin UI as a dedicated panel or table on the Withdrawal Automation tab, accessible to both Super Admin and Trust & Safety roles. The UI SHALL display all changes made within the last 12 months; changes older than 12 months are not required to be shown in the UI.
15. If a withdrawal processing error occurs after an automation setting change, the system SHALL surface an error notification to Super Admin users both within the admin UI and via a Slack notification, identifying the affected method and the nature of the failure.

### Access Control Requirements

13. Users with the **Super Admin** role SHALL have full read/write access to all automation settings.
14. Users with the **Trust & Safety** role SHALL have read-only access — they can view all settings but cannot modify thresholds or toggle pause state.
15. All other roles SHALL NOT have access to the Withdrawal Automation tab.

### Non-Functional Requirements

- All threshold and pause-state changes must be atomic — partial saves must not leave the system in an inconsistent state.
- The audit log must be append-only and not editable by any admin role.
- The tab must load and reflect the current live state of automation settings, not a cached snapshot.

## Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Time to pause a payment method during an incident | Hours (requires eng deploy) | < 2 minutes | Incident retrospectives |
| Incidents where automation continued after an intended pause | Unknown | 0 | Incident log |
| Audit log referenced in compliance/risk review | Never | ≥ 1 use within 90 days of launch | Compliance team confirmation |

## Scope

### In Scope

- Withdrawal Automation tab on the Payments admin page
- Per-method threshold control for Aeropay, Card, PayPal, Venmo
- Pause/resume toggle per method
- Skrill displayed as manual-only (no automation)
- Confirmation dialog required before any change is applied
- Last-modified audit trail per row
- In-UI audit trail panel visible to Super Admin and Trust & Safety
- Error notifications to Super Admin when withdrawal processing errors occur after a setting change
- Role-based access: Super Admin (edit), Trust & Safety (read-only)
- Audit log of all changes

### Out of Scope

- Per-user or per-geography automation rules — v1 is method-level only
- Scheduled automation pauses (e.g., "pause every Friday at 5 PM") — v1 is manual toggle only
- Surfacing automation state to end users or in the user-facing withdrawal flow
- Changing the second automation condition (single linked payment method) — v1 carries it forward as-is

## V1 Launch Thresholds

| Method  | Threshold | Automation |
|---------|-----------|------------|
| Card    | $250      | Enabled    |
| PayPal  | $500      | Enabled    |
| Venmo   | $500      | Enabled    |
| Aeropay | $1,000    | Enabled    |
| Skrill  | —         | Manual only |

## Acceptance Criteria

- Given a Super Admin is on the Payments admin page, when they navigate to the Withdrawal Automation tab, then they see a row for each of the five payment methods with the current threshold, pause state, and last-modified info.
- Given a Super Admin edits a method's threshold or toggles a pause state, when they attempt to save, then a confirmation dialog is shown summarizing the change; the change is only applied upon explicit confirmation and cancelled if dismissed.
- Given a Super Admin confirms a threshold change, when the change is applied, then new withdrawals for that method immediately use the updated threshold, and the audit log records the change with the admin's username, previous value, new value, and UTC timestamp.
- Given a Super Admin toggles the pause switch for a method, when the toggle is saved, then no new withdrawal requests for that method are auto-processed, and the row displays a "Paused" status indicator.
- Given a method is paused and a withdrawal for that method is already in `Processing` state, when the pause is applied, then that withdrawal continues to completion unaffected.
- Given a Super Admin resumes a paused method, when the toggle is saved, then automation resumes for new withdrawal requests using the method's current threshold.
- Given a Trust & Safety user is on the Withdrawal Automation tab, when the page loads, then all settings are visible but all inputs and toggles are disabled — no edits can be made.
- Given a user without Super Admin or Trust & Safety role accesses the Payments admin page, then the Withdrawal Automation tab is not visible.
- Given a withdrawal processing error occurs after an automation setting change, when the error is detected, then Super Admin users see an error notification in the admin UI and receive a Slack notification, both identifying the affected method and the nature of the failure.
- Given a Super Admin or Trust & Safety user views the Withdrawal Automation tab, then an audit trail panel is visible showing all changes made within the last 12 months with admin username, action, previous value, new value, and UTC timestamp.
- Given any admin views the Skrill row, then no threshold input or pause toggle is present — only a "Manual only" indicator is shown.
- Given a withdrawal request arrives for an account with two or more linked payment methods, when automation evaluates the request, then it is routed to the manual queue regardless of the withdrawal amount or method threshold.

## Dependencies

- **Payments admin page** — The existing Payments admin page must support adding a new tab.
- **Withdrawal processing system** — The automation engine must be able to read per-method threshold and pause state at the time a withdrawal is evaluated. Engineering must confirm the integration point.
- **Manual withdrawal processing queue** — Must be able to absorb increased volume if automation is paused across multiple methods simultaneously.
- **Role system** — Super Admin and Trust & Safety roles must be definable and enforceable at the tab and field level using the existing admin role system.
- **Audit log infrastructure** — An append-only audit log destination must exist or be created for recording automation setting changes.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Manual queue is overwhelmed if multiple methods are paused at once | Medium | High | Monitor queue depth post-launch; document an ops playbook for mass-pause scenarios |
| Threshold change takes effect on an in-flight evaluation, causing inconsistent behavior | Low | Medium | Confirm with engineering that threshold reads are atomic per withdrawal evaluation |
| Super Admin accidentally pauses the wrong method | Low | Medium | Add a confirmation step before applying a pause; show current state clearly before and after |
| Audit log is not referenced and compliance value goes unrealized | Medium | Low | Brief Compliance team on the log at launch; include it in the incident response runbook |

## Open Questions

1. What is the exact integration point in the withdrawal processing system where the automation evaluation occurs? _(Engineering to identify during technical design.)_
2. Which Slack channel should processing error notifications be sent to, and is there an existing webhook/integration to use? _(Engineering / ops to confirm.)_

## Sign-Off Log

| Role | Name | Status | Date | Notes |
|---|---|---|---|---|
| Product Lead | pfinnell | pending | — | — |
| Engineering Lead | — | pending | — | — |
| Design Lead | — | pending | — | — |
| QA Lead | — | pending | — | — |
| Finance / Risk | — | pending | — | Confirm threshold values and queue capacity |
| Compliance | — | pending | — | Confirm audit log retention requirements |

## Engineering Notes (Deferred)

<!-- Optional: populated during brainstorm when technical detail surfaces.
     Consumed by prd-expert-review and engineering handoff. Not scored by prd-review. -->

---

## Review Findings
<!-- Populated by prd-review skill -->

## Catalog Review
<!-- Populated by prd-review skill (catalog panel). -->

## Expert Assessment

### Affected Services
<!-- Populated by prd-expert-review skill -->

### Complexity Scores
<!-- Populated by prd-expert-review skill -->

### Suggested Approach
<!-- Populated by prd-expert-review skill -->
