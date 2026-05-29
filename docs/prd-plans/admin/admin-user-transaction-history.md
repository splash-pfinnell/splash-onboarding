---
type: prd
title: "Admin User Transaction History — Deposit Refunds"
slug: admin-user-transaction-history
version: 1.2.0
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

T&S and CS agents currently process deposit refunds through the Adjust Balance section of Splash Admin, which requires navigating away from the user's transaction history. This flow forces operators to manually copy and paste transaction IDs to complete a refund, and provides no visibility into which deposits have already been refunded — leaving agents to cross-reference information across two separate screens.

Refunding a deposit should be possible directly from the transaction tab, keeping the operator in context, eliminating manual ID lookups, and making refund status immediately visible on the transaction record.

## User Stories

- As a **T&S or CS agent**, I want to initiate a refund on a deposit transaction directly from the transaction history page, without switching to another tab or copying a specific transaction ID.
- As a **T&S or CS agent**, I want refunded deposits to reflect their updated status on the transaction history page immediately after a refund is processed, so that I have an accurate view of a user's transaction status.

## Requirements

### Functional Requirements

1. Deposit transactions on the Admin user detail page SHALL display a Refund action accessible to authorized operators.
2. The Refund action SHALL require an explicit confirmation step before the refund is processed.
3. Upon successful refund, the transaction row SHALL reflect the updated status (Refunded) without requiring a full page reload.
4. If a refund fails, the operator SHALL be presented with a clear error message indicating the failure reason.
5. The Refund action SHALL be gated behind a specific Admin role or permission — not visible or actionable to unauthorized roles.
6. Only deposit transactions SHALL have the Refund action — other transaction types (withdrawals, contest entries, bonus credits) SHALL NOT expose this action.
7. If a deposit is ineligible for refund (e.g. already refunded, funds already spent, payment method restriction), the Refund button SHALL be replaced with a disabled state and a brief reason label (e.g. "Already refunded", "Ineligible") — the action SHALL NOT be silently hidden.
8. A deposit that has already been successfully refunded SHALL display its status as Refunded and SHALL NOT present a Refund action.
9. The confirmation dialog SHALL default to a full refund of the deposit amount.
10. The confirmation dialog SHALL include a checkbox to enable a partial refund. When checked, an amount input field SHALL appear allowing the operator to enter a custom refund amount.
11. The partial refund amount SHALL be validated to be greater than zero and no greater than the original deposit amount before submission is allowed.

### Non-Functional Requirements

7. Every refund action SHALL produce an immutable audit log entry capturing: operator identity, timestamp, transaction ID, deposit amount, and outcome (success or failure with reason).
8. The Refund API call SHALL be idempotent — submitting the same refund request more than once SHALL NOT result in a duplicate refund.
9. The Refund button SHALL be disabled immediately upon first submission and remain disabled until a success or failure response is received, preventing double-submission.
10. If the payment processor does not respond within a configurable timeout, the operator SHALL see a timeout error and the transaction status SHALL remain unchanged.
11. Role-based access control enforcement SHALL occur server-side — the Refund action SHALL be rejected at the API level for unauthorized roles regardless of client-side visibility.

## Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Operator time to process a deposit refund | TBD | < 2 min | Support ticket handle time |
| Clear visibility into refunded deposits | None | Refunded status visible on transaction row | Operator observation / usability test |

## Scope

### In Scope

- Refund action on deposit transactions (authorized roles only)
- Confirmation dialog before refund is processed
- Full refund defaulted in the confirmation dialog, with a partial refund option via checkbox and amount input
- Updated transaction status after successful refund
- Error state handling for failed refunds
- Role-based access control for the Refund action

### Out of Scope

- Refunds on non-deposit transaction types (withdrawals, contest entries, bonus credits)
- Refunds on non-deposit transaction types beyond what is already excluded above
- Manual balance adjustments or credits
- Exporting or downloading transaction history
- Transaction display improvements (covered by Admin User Transaction Visibility PRD)

## Acceptance Criteria

- Given a deposit transaction is displayed, when an authorized operator views it, then a Refund button is present and actionable.
- Given an operator clicks Refund, when the confirmation dialog appears, then the full deposit amount is pre-filled and the operator must confirm before the refund is processed.
- Given the confirmation dialog is open, when the operator checks the partial refund checkbox, then an amount input field appears.
- Given the partial refund amount input is visible, when the operator enters an amount greater than zero and no greater than the deposit amount, then the submit button is enabled.
- Given the partial refund amount is invalid (zero, negative, or exceeds deposit), when the operator attempts to submit, then an inline validation error is shown and submission is blocked.
- Given an operator confirms the refund, when the refund is successfully processed, then the transaction row status updates to Refunded.
- Given a refund fails, when the failure occurs, then the operator sees a clear error message and the transaction status is unchanged.
- Given a user does not have sufficient balance to cover the refund amount, when the refund is submitted, then the refund fails and the operator is shown an error indicating insufficient balance.
- Given an unauthorized operator views a deposit transaction, when they view the row, then no Refund button is visible.
- Given a non-deposit transaction is displayed, when any operator views it, then no Refund button is present.
- Given a deposit is ineligible for refund, when an authorized operator views it, then a disabled Refund button with a reason label is shown (not silently hidden).
- Given a deposit has already been refunded, when any operator views it, then the transaction status shows Refunded and no Refund button is present.
- Given an operator submits a refund, when the request is in flight, then the Refund button is disabled until a response is received.
- Given a refund is initiated, then an audit log entry is created capturing the operator identity, timestamp, transaction ID, amount, and outcome.

## Dependencies

- **Splash Admin service** — User detail page must support the Refund action on deposit rows.
- **Refund API** — Engineering must identify or build the API endpoint that processes deposit refunds and returns updated transaction state. Owner TBD.
- **Payment processor** — Downstream integration with the payment processor that handles the actual refund settlement.
- **Wallet / ledger service** — Refunds likely affect wallet balance and must be reflected accurately in the ledger.
- **Admin RBAC** — Role-based access control must support a permission that gates the Refund action. Enforcement must occur server-side, not just client-side.
- **Audit log / event store** — A durable store must exist (or be created) for refund action audit entries. Owner TBD.

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
