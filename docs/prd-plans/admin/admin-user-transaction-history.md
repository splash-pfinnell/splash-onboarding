---
type: prd
title: "Admin User Transaction History — Deposit Refunds & Withdrawal Review"
slug: admin-user-transaction-history
version: 1.3.0
template_version: 1.3.0
domain: admin
product_line: internal-tool
status: draft
created: 2026-05-29
updated: 2026-11-12
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
  - withdrawals
jira: EDGE-28263
---

# Admin User Transaction History — Deposit Refunds & Withdrawal Review

## Problem Statement

T&S and CS agents currently process deposit refunds through the Adjust Balance section of Splash Admin, which requires navigating away from the user's transaction history. This flow forces operators to manually copy and paste transaction IDs to complete a refund, and provides no visibility into which deposits have already been refunded — leaving agents to cross-reference information across two separate screens.

Refunding a deposit should be possible directly from the transaction tab, keeping the operator in context, eliminating manual ID lookups, and making refund status immediately visible on the transaction record.

Withdrawals have the same problem in reverse. A withdrawal sitting in `Processing` can only be approved or rejected from the separate Withdrawal Requests page (`/withdrawal-requests`). An operator reviewing a user's history — often the exact moment they decide whether a withdrawal is legitimate — has to leave the page, find the request in the queue, and act on it there. Approve and Reject should be available on the `Processing` withdrawal row itself, using the same flow and reasons as the Withdrawal Requests page.

## User Stories

- As a **T&S or CS agent**, I want to initiate a refund on a deposit transaction directly from the transaction history page, without switching to another tab or copying a specific transaction ID.
- As a **T&S or CS agent**, I want refunded deposits to reflect their updated status on the transaction history page immediately after a refund is processed, so that I have an accurate view of a user's transaction status.
- As a **Super Admin or Trust & Safety operator (internal or external)**, I want to approve or reject a `Processing` withdrawal from the user's transaction history, so that I can act on it while I'm reviewing the user's activity instead of finding it again on the Withdrawal Requests page.
- As a **Super Admin or Trust & Safety operator**, I want to pick the same approve and reject reasons I use on the Withdrawal Requests page, so that withdrawal decisions are recorded the same way whichever page they were made from.

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

#### Withdrawal Approve / Reject

12. Withdrawal transactions in `Processing` status SHALL display a **Review** dropdown in the Action column, with two options: **Approve** and **Reject**.
13. The Review dropdown SHALL be shown only to the **Super Admin**, **Trust & Safety (Internal)** and **Trust & Safety (External)** roles. All other roles SHALL see no action on the row (the column shows "—").
14. Withdrawals in any status other than `Processing` (Completed, Failed, Canceled, Approved, Rejected) SHALL NOT show the Review dropdown.
15. Selecting Approve or Reject SHALL open the same confirmation dialog used on the Withdrawal Requests page: "Are you sure you want to approve/reject this request for {USER NAME} ({amount})?", with a required **Reason** select.
16. The approve reasons SHALL match the Withdrawal Requests page:
    - Withdrew winnings
    - Withdrew funds/winnings after contest entry
    - Withdrew funds + winnings
    - Withdrew deposit, no contest entry
    - Withdrew duplicate deposit
    - Withdrew funds after canceling Contest entry
    - Withdrew funds after Contest/QP entry was canceled
    - Withdrew commish rewards
    - Withdrew Marketplace Sales
    - Other
17. The reject reasons SHALL match the Withdrawal Requests page:
    - Canceled - Refunding deposit
    - Canceled - Suspicious activity / Needs Review
    - Canceled - Failed withdrawal
    - Canceled - Pushing to ACH
    - User requesting to cancel withdrawal
    - Other
18. The reason lists SHALL come from the same source as the Withdrawal Requests page, so that adding or renaming a reason there changes it here too. They SHALL NOT be a separate copy.
19. The confirm button SHALL stay disabled until a reason is selected. If "Other" is selected, a free-text field SHALL appear and SHALL be required.
20. Approving or rejecting from the transaction history SHALL call the same backend action as the Withdrawal Requests page and SHALL have the same downstream effects (payout release on approve; cancellation and return of funds on reject).
21. On success, the withdrawal row SHALL update its status and show the selected reason in the Reason column without a full page reload, and the Review dropdown SHALL be replaced by who took the action (e.g. "Approved by {operator}").
22. A withdrawal approved or rejected from either page SHALL show as actioned on the other.

### Non-Functional Requirements

1. Every refund action SHALL produce an immutable audit log entry capturing: operator identity, timestamp, transaction ID, deposit amount, and outcome (success or failure with reason).
2. The Refund API call SHALL be idempotent — submitting the same refund request more than once SHALL NOT result in a duplicate refund.
3. The Refund button SHALL be disabled immediately upon first submission and remain disabled until a success or failure response is received, preventing double-submission.
4. If the payment processor does not respond within a configurable timeout, the operator SHALL see a timeout error and the transaction status SHALL remain unchanged.
5. Role-based access control enforcement SHALL occur server-side — the Refund action SHALL be rejected at the API level for unauthorized roles regardless of client-side visibility.
6. Withdrawal Approve / Reject SHALL be enforced server-side for the Super Admin and Trust & Safety (Internal and External) roles only. A request from any other role SHALL be rejected at the API, whatever the UI shows.
7. If a withdrawal is no longer `Processing` when the operator submits (for example it was already actioned on the Withdrawal Requests page or by automation), the action SHALL fail with a clear "This withdrawal has already been actioned" message, and the row SHALL refresh to its current status. It SHALL NOT be processed twice.
8. Every withdrawal approve or reject SHALL produce the same audit record as the Withdrawal Requests page (operator, timestamp, transaction ID, amount, action, reason, and note), plus the page it was taken from.

## Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Operator time to process a deposit refund | TBD | < 2 min | Support ticket handle time |
| Clear visibility into refunded deposits | None | Refunded status visible on transaction row | Operator observation / usability test |
| Share of `Processing` withdrawals actioned from the transaction history | 0% | TBD after launch | Audit log, by source page |
| Time from opening a user's history to actioning their withdrawal | TBD | < 1 min | Audit log timestamps |

## Scope

### In Scope

- Refund action on deposit transactions (authorized roles only)
- Confirmation dialog before refund is processed
- Full refund defaulted in the confirmation dialog, with a partial refund option via checkbox and amount input
- Updated transaction status after successful refund
- Error state handling for failed refunds
- Role-based access control for the Refund action
- Review dropdown (Approve / Reject) on `Processing` withdrawal rows, matching the Withdrawal Requests page flow and reasons
- Role gating for withdrawal review: Super Admin and Trust & Safety (Internal and External) only

### Out of Scope

- Refunds on non-deposit transaction types (withdrawals, contest entries, bonus credits)
- Changes to the Withdrawal Requests page itself, or to its reason lists
- Bulk approve or reject from the transaction history
- Approving or rejecting withdrawals in any status other than `Processing`
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

**Withdrawal Approve / Reject**

- Given a withdrawal in `Processing` status, when a Super Admin or Trust & Safety (Internal or External) operator views it, then a Review dropdown with Approve and Reject is shown in the Action column.
- Given a withdrawal in `Processing` status, when an operator in any other role views it, then no Review dropdown is shown.
- Given a withdrawal in any other status, when any operator views it, then no Review dropdown is shown.
- Given an operator selects Approve, when the dialog opens, then it reads "Are you sure you want to approve this request for {USER NAME} ({amount})?" and the Reason list matches the Withdrawal Requests page approve reasons.
- Given an operator selects Reject, when the dialog opens, then it reads "Are you sure you want to reject this request for {USER NAME} ({amount})?" and the Reason list matches the Withdrawal Requests page reject reasons.
- Given the dialog is open, when no reason is selected, or "Other" is selected with no text, then the confirm button is disabled.
- Given an operator confirms an approve or reject, when it succeeds, then the row status, Reason column and Action column update without a page reload.
- Given the withdrawal was already actioned elsewhere, when the operator confirms, then they see "This withdrawal has already been actioned", the row refreshes to its current status, and nothing is processed twice.
- Given a withdrawal is approved or rejected from the transaction history, when the Withdrawal Requests page is viewed, then that request shows as actioned with the same reason.
- Given a non-permitted role calls the approve/reject API directly, then the request is rejected.

## Dependencies

- **Splash Admin service** — User detail page must support the Refund action on deposit rows.
- **Refund API** — Engineering must identify or build the API endpoint that processes deposit refunds and returns updated transaction state. Owner TBD.
- **Payment processor** — Downstream integration with the payment processor that handles the actual refund settlement.
- **Wallet / ledger service** — Refunds likely affect wallet balance and must be reflected accurately in the ledger.
- **Admin RBAC** — Role-based access control must support a permission that gates the Refund action. Enforcement must occur server-side, not just client-side.
- **Audit log / event store** — A durable store must exist (or be created) for refund action audit entries. Owner TBD.
- **Withdrawal Requests page / withdrawal review API** — The transaction history must reuse the existing approve/reject action, reasons source and audit trail behind `/withdrawal-requests`, not a parallel implementation.
- **Admin roles** — Super Admin, Trust & Safety (Internal) and Trust & Safety (External) must be distinguishable server-side to gate withdrawal review.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Refund action triggers downstream side effects (wallet, ledger, payment processor) that are not fully mapped | Medium | High | Engineering spike to trace the full refund call chain before implementation begins |
| Unauthorized refund initiation if RBAC is not enforced | Low | High | Gate Refund action behind a specific Admin permission; confirm with Auth/Trust & Safety |
| Duplicate refund submissions if operator double-clicks or retries | Medium | High | Enforce idempotency at the API layer; disable the Refund button after first submission |
| Not all deposits are eligible for refund (already-spent funds, old deposits, payment method restrictions) | Medium | Medium | Confirm eligibility rules with Finance/Engineering; display clear ineligibility state if refund is not available |
| Two operators (or automation) action the same withdrawal from different pages at once | Medium | High | Server checks the withdrawal is still `Processing` at submit time; the second request fails with "already actioned" |
| Reason lists drift between the two pages | Medium | Low | Load reasons from the same source as the Withdrawal Requests page |
| External T&S operators get broader withdrawal access than intended | Low | High | Confirm with T&S leadership that External T&S should approve as well as reject; enforce server-side |

## Open Questions

1. What is the full downstream call chain for a deposit refund — wallet service, payment processor, ledger? Who owns the refund API?
2. Which Admin roles should be permitted to initiate a deposit refund? Is there an existing permission that fits, or does a new one need to be created?
3. Are all deposits refundable, or are there eligibility rules (e.g. already-spent funds, age of deposit, payment method)?
4. Should the refund action require a manager approval step, or is single-operator confirmation sufficient?
5. What audit log record is created when a refund is initiated? Where is it stored?
6. Does this PRD depend on the visibility improvements in the Admin User Transaction Visibility PRD, or can it ship independently against the existing transaction UI?
7. After an approve, should the row show `Approved` until the payout settles, or go straight to the processor's status? (The demo shows `Approved`.)
8. Should withdrawal review and deposit refunds share one role list? Withdrawal review is limited to Super Admin and T&S (Internal and External); refund roles are still open (question 2).
9. Does the existing approve/reject endpoint behind `/withdrawal-requests` already enforce these roles, or does that need adding?

## Sign-Off Log

| Role | Name | Status | Date | Notes |
|---|---|---|---|---|
| Product Lead | pfinnell | pending | — | — |
| Engineering Lead | — | pending | — | — |
| Design Lead | — | pending | — | — |
| QA Lead | — | pending | — | — |
| Trust & Safety | — | pending | — | Confirm withdrawal review roles, including External T&S |

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
