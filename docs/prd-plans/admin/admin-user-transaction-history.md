---
type: prd
title: "Admin User Transaction History — Deposit Refunds"
slug: admin-user-transaction-history
version: 1.1.0
template_version: 1.3.0
domain: admin
product_line: internal-tool
status: draft
created: 2026-05-29
updated: 2026-05-29
author: pfinnell
source: brainstorm
target_repo: splash-admin
source_one_pager: admin/admin-user-transaction-history-1pager.md
related_prds:
  - admin/admin-user-transaction-visibility.md
status_history:
  - status: draft
    date: 2026-05-29
review_passes: []
tags:
  - admin
  - users
  - transactions
  - finance
  - customer-support
---

# Admin User Transaction History — Deposit Refunds

## Problem Statement

Customer Support operators resolving deposit disputes currently have no way to initiate a refund from Splash Admin. They must escalate to engineering or switch to a separate payment tool to process the refund, then manually verify the outcome back in Admin. This adds unnecessary handling time and engineering load for a routine support action.

## User Stories

- As a **Customer Support operator**, I want to initiate a refund on a deposit transaction directly from the transaction history page, so that I can resolve deposit disputes without escalating to engineering or switching to another tool.
- As a **Finance operator**, I want refunded deposits to reflect their updated status on the transaction history page immediately after a refund is processed, so that I have an accurate view of a user's transaction state.

## Requirements

### Functional Requirements

1. Deposit transactions on the Admin user detail page SHALL display a Refund action accessible to authorized operators.
2. The Refund action SHALL require an explicit confirmation step before the refund is processed.
3. Upon successful refund, the transaction row SHALL reflect the updated status (Reversed) without requiring a full page reload.
4. If a refund fails, the operator SHALL be presented with a clear error message indicating the failure reason.
5. The Refund action SHALL be gated behind a specific Admin role or permission — not visible or actionable to unauthorized roles.
6. Only deposit transactions SHALL have the Refund action — other transaction types (withdrawals, contest entries, bonus credits) SHALL NOT expose this action.

### Non-Functional Requirements

_To be defined. Consider: audit logging for all refund actions (who initiated, when, amount), idempotency to prevent duplicate refund submissions, timeout handling if the payment processor is slow, and role-based access control integration._

## Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Engineering escalations for deposit refund processing | TBD | 0 | Escalation tracking |
| Operator time to process a deposit refund | TBD | < 2 min | Support ticket handle time |

## Scope

### In Scope

- Refund action on deposit transactions (authorized roles only)
- Confirmation dialog before refund is processed
- Updated transaction status after successful refund
- Error state handling for failed refunds
- Role-based access control for the Refund action

### Out of Scope

- Refunds on non-deposit transaction types (withdrawals, contest entries, bonus credits)
- Partial refunds — full deposit refund only in v1
- Manual balance adjustments or credits
- Exporting or downloading transaction history
- Transaction display improvements (covered by Admin User Transaction Visibility PRD)

## Acceptance Criteria

- Given a deposit transaction is displayed, when an authorized operator views it, then a Refund button is present and actionable.
- Given an operator clicks Refund, when the confirmation dialog appears, then the operator must confirm before the refund is processed.
- Given an operator confirms the refund, when the refund is successfully processed, then the transaction row status updates to Reversed.
- Given a refund fails, when the failure occurs, then the operator sees a clear error message and the transaction status is unchanged.
- Given an unauthorized operator views a deposit transaction, when they view the row, then no Refund button is visible.
- Given a non-deposit transaction is displayed, when any operator views it, then no Refund button is present.

## Dependencies

- **Splash Admin service** — User detail page must support the Refund action on deposit rows.
- **Refund API** — Engineering must identify or build the API endpoint that processes deposit refunds and returns updated transaction state. Owner TBD.
- **Payment processor** — Downstream integration with the payment processor that handles the actual refund settlement.
- **Wallet / ledger service** — Refunds likely affect wallet balance and must be reflected accurately in the ledger.
- **Admin RBAC** — Role-based access control must support a permission that gates the Refund action.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Refund action triggers downstream side effects (wallet, ledger, payment processor) that are not fully mapped | Medium | High | Engineering spike to trace the full refund call chain before implementation begins |
| Unauthorized refund initiation if RBAC is not enforced | Low | High | Gate Refund action behind a specific Admin permission; confirm with Auth/Trust & Safety |
| Duplicate refund submissions if operator double-clicks or retries | Medium | High | Enforce idempotency at the API layer; disable the Refund button after first submission |
| Not all deposits are eligible for refund (already-spent funds, old deposits, payment method restrictions) | Medium | Medium | Confirm eligibility rules with Finance/Engineering; display clear ineligibility state if refund is not available |

## Open Questions

1. What is the full downstream call chain for a deposit refund — wallet service, payment processor, ledger? Who owns the refund API?
2. Which Admin roles should be permitted to initiate a deposit refund? Is there an existing permission that fits, or does a new one need to be created?
3. Are all deposits refundable, or are there eligibility rules (e.g. already-spent funds, age of deposit, payment method)?
4. Should the refund action require a manager approval step, or is single-operator confirmation sufficient?
5. What audit log record is created when a refund is initiated? Where is it stored?
6. Does this PRD depend on the visibility improvements in the Admin User Transaction Visibility PRD, or can it ship independently against the existing transaction UI?

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
