---
type: prd
title: "Admin User Payment Methods"
slug: admin-user-payment-methods
version: 1.2.0
template_version: 1.3.0
domain: admin
product_line: internal-tool
status: draft
created: 2026-08-27
updated: 2026-08-27
author: pfinnell
source: brainstorm
target_repo: splash-admin
source_one_pager: admin/admin-user-payment-methods-1pager.md
related_prds:
  - admin/admin-user-transaction-history.md
status_history:
  - status: draft
    date: 2026-08-27
  - status: draft
    date: 2026-08-27
review_passes: []
tags:
  - admin
  - users
  - payments
  - customer-support
  - trust-and-safety
---

# Admin User Payment Methods

## Problem Statement

CS, T&S, and Finance operators have no visibility into, or control over, which payment methods are linked to a user's account from within Splash Admin. Answering basic questions — does this user have more than one payment method linked, is their card still active or was it removed, which method did a given deposit come from — currently requires escalating to Payments engineering or piecing together partial answers from transaction history, which shows the payment type used per transaction but not the current state of the underlying method.

Beyond visibility, operators have no lever to act on a method that needs to go away. A card with chargeback risk, a compromised or disputed account, or a method a user asks to have removed all require an engineering ticket today. Operators need to be able to remove a payment method from the account entirely, directly from Admin.

This feature is a standalone payment-method management surface. It is intentionally decoupled from withdrawal automation's per-method thresholds, pause toggles, and eligibility rules — those remain a separate system owned by the Withdrawal Automation PRD. Removing a method here means it can no longer be used at all; it is not an automation-routing decision.

## User Stories

- As a **CS or T&S operator**, I want to see all payment methods linked to a user's account on their Admin detail page, so that I can answer support questions about a user's payment methods without leaving Admin or filing an engineering request.
- As a **CS operator**, I want to remove a payment method from a user's account when the user requests it or it's no longer valid, so that it can no longer be used going forward.
- As a **T&S operator** investigating fraud or a chargeback pattern, I want to remove a specific payment method from an account, so that I can stop its use without waiting on engineering.
- As a **Finance operator**, I want to see when a payment method was linked and when it was last used, so that I can assess account risk during an investigation.

## Requirements

### Functional Requirements

1. The system SHALL add a "Payment Methods" tab to the Admin user detail page, alongside existing tabs (Details, Transactions, etc.).
2. The Payment Methods tab SHALL list every payment method linked to the account, showing: method type (Aeropay, Card, PayPal, Venmo, Skrill), masked identifier, status, date added, and date last used.
3. An authorized operator SHALL be able to remove a payment method from the account. Removal SHALL require an explicit confirmation step before it applies.
4. A removed payment method SHALL NOT be permanently deleted from the record — it SHALL be marked Removed and retained for historical/audit purposes.
5. Removed methods SHALL be hidden from the default view and only shown when the operator enables a "Show removed methods" toggle.
6. A removed method SHALL NOT expose a removal control — it is a terminal state in v1 (no restore action).
7. The tab SHALL display an empty state when a user has no linked payment methods (excluding removed, unless the toggle is enabled).
8. Every masked identifier SHALL show only the last 4 digits or characters of the underlying value — never the full card, account, or email identifier.
9. This feature SHALL NOT read from or write to withdrawal automation's per-method threshold, pause state, or Pause All controls. Removal here is independent of automation eligibility.

### Non-Functional Requirements

1. No unmasked payment credential (full card number, full bank account number, full email) SHALL ever be transmitted to or rendered by Admin.
2. A removal SHALL take effect in downstream deposit/withdrawal flows within a defined SLA (target: near-real-time; confirmed with Payments engineering).
3. Every removal SHALL produce an immutable audit log entry capturing: operator identity, timestamp, payment method ID, and action type.
4. Access to view the Payment Methods tab, and access to edit (remove), SHALL be gated by separate Admin permissions — view access does not imply edit access. Enforcement SHALL occur server-side.
5. The Remove control SHALL be disabled while a removal is in flight and re-enabled only after a success or failure response, preventing duplicate submissions.
6. Fetching payment method data SHALL not add more than ~1 second to user detail page load at P95.

## Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Operator escalations to Payments engineering for payment-method lookups or removal requests | Unmeasured, recurring | Near zero | Support ticket / escalation channel review |
| Time for an operator to look up or remove a payment method | Requires engineering ticket (hours to days) | < 1 minute | Operator observation / usability test |

## Scope

### In Scope

- Payment Methods tab on the Admin user detail page
- Method type, masked identifier, status, date added, and last used date per method
- Operator control to remove a payment method, with confirmation
- Removed methods retained and shown only via a "Show removed methods" toggle
- Audit logging of removals

### Out of Scope

- Restricting a payment method to deposits only or withdrawals only, or any per-method usage-mode setting
- Adding a new payment method from Admin
- Unmasking or displaying full card/account/email identifiers
- Restoring a removed/deleted payment method
- Initiating deposits or withdrawals from this tab
- Any integration with, or dependency on, withdrawal automation's per-method thresholds, pause toggles, or Pause All controls — that remains a separate system

## Acceptance Criteria

- Given a user has one or more linked payment methods, when an operator opens the Payment Methods tab, then each method's type, masked identifier, status, date added, and last used date are shown.
- Given an authorized operator removes a payment method, when they confirm the removal, then the method's status updates to Removed and it drops out of the default (non-removed) view.
- Given an operator attempts to remove a payment method, when the confirmation step appears, then the removal is not applied until they explicitly confirm.
- Given a payment method has been removed, when the operator enables "Show removed methods," then the method reappears marked Removed, with no removal control available.
- Given a user has no non-removed linked payment methods, when the tab is opened, then an empty state is shown.
- Given any payment method identifier, when rendered anywhere in Admin, then only the last 4 digits or characters are visible and the remainder is masked.
- Given an operator without edit permission, when they view the tab, then the removal control is not available to them (view-only), and given an operator without view permission, then the tab is not accessible at all — both enforced server-side.
- Given a removal is submitted, when the request is in flight, then the Remove control is disabled until a response is received.
- Given a removal occurs, then an audit log entry is created capturing the operator identity, timestamp, method, and action type.

## Dependencies

- **Payments service** — Source of truth for linked payment methods and their status. Must support both reading all linked methods (across Aeropay, Card, PayPal, Venmo, Skrill) and writing a removal per method.
- **Splash Admin service** — User detail page must support a new tab with a removal control, not just a read-only view.
- **Admin RBAC** — Separate view and edit permissions gating this tab, consistent with the existing Transactions permission model.
- **Audit log / event store** — A durable store for removal audit entries. Owner TBD.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Payments service may not support writing a removal per method today — only reads may exist | Medium | High | Engineering spike with Payments to confirm read/write contract before implementation |
| A removal doesn't propagate to deposit/withdrawal flows quickly enough, creating a window where a removed method is still usable | Medium | High | Confirm propagation SLA with Payments engineering; surface a pending/in-flight state in the UI if propagation isn't instant |
| Operators without a clear mental model may confuse removal here with withdrawal automation's per-method pause controls | Medium | Medium | Keep the two features visually and functionally separate in Admin; do not share UI components, data, or terminology (e.g. avoid "Paused") between them |
| PCI scope creep if masking is not enforced server-side | Low | High | Masking must happen server-side; Admin should never receive full identifiers in the first place |

## Open Questions

1. What is the API contract for reading and writing linked payment methods (Aeropay, Card, PayPal, Venmo, Skrill) for a given user? Who owns it?
2. What is the propagation SLA between an Admin-initiated removal and that change taking effect in live deposit/withdrawal flows?
3. Should view access and edit access be two distinct RBAC permissions, or one combined permission?
4. Is a removal reversible at the data layer (even though there's no restore action in v1), in case a removal needs to be corrected by engineering?
5. Should there be a required reason/note field when an operator removes a method, for audit clarity?

## Sign-Off Log

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
