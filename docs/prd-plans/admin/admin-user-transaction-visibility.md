---
type: prd
title: "Admin User Transaction Visibility"
slug: admin-user-transaction-visibility
version: 1.0.0
template_version: 1.3.0
domain: admin
product_line: internal-tool
status: draft
created: 2026-05-29
updated: 2026-05-29
author: pfinnell
source: brainstorm
target_repo: splash-admin
source_one_pager: admin/admin-user-transaction-visibility-1pager.md
status_history:
  - status: draft
    date: 2026-05-29
review_passes: []
tags:
  - admin
  - users
  - transactions
  - customer-support
---

# Admin User Transaction Visibility

## Problem Statement

Customer Support and Trust and Safety operators investigate transaction-related tickets daily, but the transaction history view in Splash Admin is too hard to read. Key fields have no visual hierarchy, and it's hard to distinguish one transaction category from another.

Scanning a user's history to find all deposits, or all failed transactions, requires reading every row individually. A task that should take seconds takes minutes of manual interpretation, slowing ticket resolution and account review.

## User Stories

- As a **CS or T&S agent**, I want to see a user's transaction history in a clear, scannable format, so that I can quickly understand what happened to a user's account without using Metabase.
- As a **CS or T&S agent**, I want transaction rows color-coded by type, so that I can visually scan a user's history and locate the relevant transaction category instantly without reading every row.
- As a **Finance operator**, I want to see type, amount, status, and timestamp at a glance for each transaction, so that I can investigate balance discrepancies without leaving the Admin user detail page.

## Requirements

### Functional Requirements

1. The system SHALL display a Transaction History section on the Splash Admin user detail page listing all transactions for that user.
2. Each transaction row SHALL display at a glance: human-readable transaction type label, amount, status badge, and timestamp.
3. Transaction status SHALL be displayed as a colored status badge with one of the following states: Completed, Pending, Failed, Reversed.
4. Transactions SHALL be displayed in reverse-chronological order (newest first).
5. Each transaction row SHALL be color-coded by transaction type per the following assignments:

   | Transaction Type | Row Color |
   |---|---|
   | Deposit | Green |
   | Entry | Blue |
   | Slip Entry | Light Blue |
   | Payout | Gold |
   | Slip Payout | Yellow |
   | Contest / Entry Canceled | Orange |
   | Withdrawal | Red |
   | Bonus | Teal (Splash Sports brand teal) |
6. Each transaction row SHALL support expansion to reveal secondary fields: transaction ID, linked slip or contest ID (where applicable), and any additional diagnostic fields confirmed with engineering.

### Non-Functional Requirements

_To be defined. Consider: access control (which Admin roles can view transaction history), data freshness (real-time vs. cached), performance impact of loading full transaction history for high-volume users, and PII/financial data handling._

## Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Metabase transaction queries run by CS / T&S | TBD | Reduced | Metabase query logs |
| Operator time to identify a transaction type/status | TBD | < 5 sec | Operator observation / usability test |
| Support tickets requiring Admin-to-external-tool context switch | TBD | Reduced | Operator survey / ticket notes |

## Scope

### In Scope

- Redesigned transaction history section on the Admin user detail page
- Human-readable transaction type labels replacing internal codes
- Colored status badges (Completed, Pending, Failed, Reversed)
- Color-coded row styling — one distinct color per transaction type category
- Expandable transaction rows with secondary fields (transaction ID, linked slip/contest ID)
- Reverse-chronological ordering
- All transaction types present in the Splash data model (inventory to be confirmed with engineering)

### Out of Scope

- Deposit refunds or any other write operations (covered by a separate PRD)
- Transaction search, filtering, or export
- Any non-user-detail-page surfaces in Admin

## Acceptance Criteria

- Given a user has transactions, when an admin operator opens that user's detail page, then a Transaction History section is visible showing all transactions.
- Given a transaction row is displayed, when the operator views it, then type label, amount, status badge, and timestamp are all visible without expanding the row.
- Given a transaction type is an internal code or enum, when it is displayed, then a human-readable label is shown instead (e.g. "Deposit" not "DEPOSIT_TXN").
- Given a transaction has a status, when it is displayed, then the status badge reflects one of: Completed, Pending, Failed, Reversed — with distinct visual styling per state.
- Given two transactions of different types are displayed, when the operator views them, then each row uses its designated color (Deposit=Green, Entry=Blue, Slip Entry=Light Blue, Payout=Gold, Slip Payout=Yellow, Contest/Entry Canceled=Orange, Withdrawal=Red, Bonus=Teal).
- Given the same transaction type appears in multiple rows, when displayed, then all rows of that type share the same color.
- Given transactions exist, when the Transaction History section loads, then entries are ordered newest-first.
- Given an operator clicks to expand a transaction row, then secondary fields (transaction ID, linked slip/contest ID) are revealed.
- Given a user has no transactions, when an admin opens their detail page, then the Transaction History section shows an empty state (not an error).

## Dependencies

- **Splash Admin service** — The existing user detail page must support the redesigned Transaction History section.
- **Transaction data model** — Engineering must inventory all transaction types and statuses to confirm the complete label mapping and the full set of color assignments.
- **Admin design system** — Design must confirm which color tokens are available for row-level type coding and how they integrate with existing badge/tag conventions.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Transaction type/status enum set is larger or more complex than expected | Medium | Medium | Engineering spike to inventory all states before design begins; confirm label mapping covers every case |
| Admin design system color tokens don't cleanly map to the defined row colors (Green, Light Blue, Gold, Yellow, Orange, Red, White) | Low | Low | Color intent is decided; Design to select closest available tokens and confirm with PM before implementation |
| Some transaction types lack a clear human-readable equivalent | Low | Low | Define a fallback label convention (e.g. title-case the enum value) for any unmapped types |
| High-volume users have thousands of transactions, causing page performance issues | Low | Medium | Paginate or lazy-load the Transaction History section if counts are high |

## Open Questions

1. What is the complete set of transaction types in the Splash data model? Who owns the mapping to human-readable labels?
2. What is the complete set of transaction statuses? Do Completed, Pending, Failed, Reversed cover every case, or are there additional states?
3. Are there transaction types that should be hidden from certain Admin roles (e.g. internal adjustment entries)?
4. What specific color tokens in the Admin design system map to Green, Light Blue, Gold, Yellow, Orange, Red, and Teal (Splash Sports brand) for row-level coding? Engineering should use the brand teal token for Bonus rather than a custom value.
6. What secondary fields are most useful to operators on expansion — transaction ID, linked contest/slip, fee breakdown?
7. Should very old transactions (e.g. > 1 year) be paginated or excluded by default?

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
