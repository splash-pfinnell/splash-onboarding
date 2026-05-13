---
type: prd
title: "Admin User Verification Attempts"
slug: admin-user-verification-attempts
version: 1.0.0
template_version: 1.3.0
domain: admin
product_line: internal-tool
status: draft
created: 2026-05-13
updated: 2026-05-13
author: pfinnell
source: brainstorm
target_repo: splash-admin
source_one_pager: admin/admin-user-verification-attempts-1pager.md
status_history:
  - status: draft
    date: 2026-05-13
review_passes: []
tags:
  - admin
  - users
  - verification
  - trust-and-safety
---

# Admin User Verification Attempts

## Problem Statement

<!-- seeded from one-pager (Why now) — expand with strategic context -->
Support and Trust & Safety teams currently split their workflow across two tools — Splash Admin and Socure — just to investigate a single user's verification history. Consolidating verification attempt data into Admin reduces context-switching, speeds up investigations, and gives Trust & Safety visibility into suspicious attempt patterns they currently can't see until they manually look up the user in Socure.

<!-- seeded from one-pager (Problem) — expand with concrete pain detail -->
Splash Admin only surfaces a user's latest verification attempt and a pass/fail status — full attempt history and failure reasons are not visible. Operators cannot see all verification attempts, the data the user submitted, or why individual attempts failed without leaving Admin and querying Socure directly.

## User Stories

<!-- seeded from one-pager (Who) — expand into "As a [role], I want..." stories -->
Splash Admin users on the Customer Support and Trust & Safety teams.

- As a **Customer Support operator**, I want to see all of a user's verification attempts on their Admin detail page, so that I can resolve verification-related support tickets without switching to Socure.
- As a **Trust & Safety operator**, I want to see the data submitted in each verification attempt and the reason it failed, so that I can identify suspicious patterns without manually querying Socure.

## Requirements

### Functional Requirements

<!-- seeded from one-pager (Solution sketch) — bullets are not testable yet; rewrite as numbered SHALL/SHOULD/MAY requirements -->

1. The system SHALL display a Verification section on the Splash Admin user detail page showing all historical verification attempts for that user.
2. Each attempt SHALL display the data submitted by the user and the reason for failure (where applicable).
3. Attempts SHALL be displayed in reverse-chronological order.
4. The Verification section SHALL be read-only — no write operations in v1.

### Non-Functional Requirements

_To be defined. Consider: access control (who can view verification data), data freshness (how often Socure data is synced or fetched), performance (page load impact of fetching attempt history), and PII handling for submitted user data._

## Success Metrics

<!-- seeded from one-pager (Success signals) — add baselines, targets, and measurement methods -->
Support and Trust & Safety teams currently perform hundreds of Socure lookups per day to investigate user verification attempts. Success means that number drops to zero — all verification history is available directly in Admin, eliminating the need to leave the tool.

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Socure lookups per day by Support/T&S | ~hundreds/day | 0 | Socure access logs |

## Scope

### In Scope

- Verification attempt history display on the Admin user detail page
- Submitted user data per attempt
- Failure reasons per failed attempt
- Reverse-chronological ordering of attempts

### Out of Scope

<!-- seeded from one-pager (Out of scope) -->
Users who have not attempted or completed verification. This feature only surfaces data for users with existing verification attempts in Socure.

Manual triggering, re-triggering, or admin override of verification status is deferred to a future iteration.

## Acceptance Criteria

_To be defined. Suggested starting points:_

- Given a user has made one or more verification attempts, when an admin operator opens that user's detail page, then a Verification section is visible showing all attempts.
- Given a verification attempt failed, when the attempt is displayed, then the failure reason is shown alongside the submitted data.
- Given attempts exist, when the Verification section loads, then attempts are ordered newest-first.
- Given a user has no verification attempts, when an admin opens their detail page, then the Verification section is not shown (or shows an empty state).

## Dependencies

- **Socure API** — Verification attempt data is sourced from Socure (`dashboard.socure.com/#/transactions`). API availability, authentication, rate limits, and the shape of transaction data must be confirmed with engineering.
- **Splash Admin service** — The existing user detail page must support adding a new Verification section.

## Risks & Mitigations

<!-- seeded from one-pager (Risks & unknowns) — expand each into risk/likelihood/impact/mitigation -->

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Socure API shape or availability is not suitable for direct integration | Medium | High | Spike to confirm API contract before implementation begins |
| Submitted user data in Socure responses contains sensitive PII requiring access gating | Medium | Medium | Confirm PII classification early; add role-based access control if required |
| Volume of attempts per user causes page performance issues | Low | Medium | Paginate or lazy-load the Verification section if attempt counts are high |

## Open Questions

<!-- seeded from one-pager (Risks & unknowns) — questions that need answers before/during implementation -->

1. What is the Socure API contract for fetching all verification attempts for a given user? Who owns this integration?
2. Are there rate limits on Socure lookups that could affect real-time display in Admin?
3. What PII is present in the submitted user data fields? Does it require access-level gating in Admin?
4. Should the Verification section be visible to all Admin roles, or only Trust & Safety?

## Sign-Off Log

<!-- Auto-populated from dossier defaults. Update names and dates as sign-offs occur. -->

| Role | Name | Status | Date | Notes |
|---|---|---|---|---|
| Product Lead | pfinnell | pending | — | — |
| Engineering Lead | — | pending | — | — |
| Design Lead | — | pending | — | — |
| QA Lead | — | pending | — | — |

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
