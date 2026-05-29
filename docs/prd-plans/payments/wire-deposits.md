---
type: prd
title: "Wire Deposits"
slug: wire-deposits
version: 1.0.0
template_version: 1.3.0
domain: payments
product_line: consumer
status: draft
created: 2026-05-14
updated: 2026-05-14
author: pfinnell
source: brainstorm
target_repo: splash-app
source_one_pager: payments/wire-deposits-1pager.md
status_history:
  - status: draft
    date: 2026-05-14
review_passes: []
tags:
  - payments
  - deposits
  - wire-transfer
  - vip
---

# Wire Deposits

## Problem Statement

The Splash Sports deposit page offers no wire transfer option. Users who want to deposit $10,000 or more have no in-app pathway to do so — they must contact support or use workarounds. This creates unnecessary friction for high-value players and limits Splash's ability to grow large-deposit volume. Wire transfers are the industry-standard method for moving large sums between financial institutions and are an expected capability for a platform serving serious sports bettors.

## User Stories

- As a **verified Splash user**, I want to see a Wire Transfer option at the bottom of the deposit page, so that I can make large deposits ($10,000+) without having to contact support.
- As a **verified Splash user**, I want to view complete wire transfer instructions in the app and close the screen when done, so that I can initiate a wire from my bank without leaving the Splash platform.
- As a **customer support operator**, I want users to find wire instructions on their own in the app, so that I spend less time manually providing wire details to high-value players.

## Requirements

### Functional Requirements

1. The system SHALL display a Wire Transfer option at the bottom of the deposit page for all logged-in users located in an allowed state.
2. When a user selects the Wire Transfer option, the system SHALL display a read-only wire instructions screen containing all of the following:
   a. Minimum wire amount: **$10,000.00** — with a note that wires under this amount will be returned and users should use Aeropay (Online Banking) for smaller deposits.
   b. A notice that the name on the user's bank account must match the name on their Splash Sports account, displaying the user's Splash Sports account name.
   c. Deadline: wires must be received Monday–Friday by 4:00 PM EST to be posted the same day.
   d. Location notice: the user must be in a valid (allowed) state for credit to be posted to their account.
   e. Splash Sports bank account details: Receiving Bank Name, Receiving Bank Address, ABA/Routing Number, Account Number, Beneficiary Bank Account Title, Beneficiary Address.
   f. Reference field value: the user's Splash Sports account email.
   g. Restriction notice: US banks only.
   h. Closing instruction: "To ensure proper posting, please adhere to the exact wire transfer instructions provided above."
3. The wire instructions screen SHALL provide a way for the user to close/dismiss the screen and return to the deposit page. No further action is required from the user within the app.
4. Wires received are reviewed and approved or denied by the VP of Finance — no automated wire matching occurs in v1.
5. The Wire Transfer option SHALL NOT be visible to users who are not logged in or who are located in a restricted state.

### Non-Functional Requirements

_To be defined. Consider: security review of how bank account details are stored and served (mock data used in development; production values provided by finance before launch), PII handling for the user's account name and email displayed in instructions, and performance impact of the wire instructions screen load._

## Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Wire deposit volume ($ total) | $0 (no in-app option) | TBD with finance | Finance/ops wire reconciliation report |
| Wire deposit count per month | 0 | TBD | Ops log |
| Support contacts re: large deposit instructions | Baseline from support tickets | ↓ 80% within 60 days of launch | Support ticket tags |
| 30-day re-deposit rate for users depositing $5k+ | Baseline TBD | Improve vs. control | Analytics |

## Scope

### In Scope

- Wire Transfer entry point at the bottom of the deposit page (all logged-in users in allowed states)
- Read-only wire instructions screen with all required bank and procedural details
- Dynamic display of the user's Splash Sports account name and email in the instructions
- Dismiss/close action returning the user to the deposit page
- Location-based visibility gating (allowed states only)

### Out of Scope

- Automated wire matching or reconciliation (v1 relies on the Reference/email field and manual ops review)
- International banks — US banks only in v1
- In-app wire initiation, cancellation, or status tracking — users initiate through their own bank
- Sub-$10,000 wire support — those users are directed to Aeropay

## Acceptance Criteria

- Given a logged-in user in an allowed state is on the deposit page, when the page loads, then a Wire Transfer option is visible at the bottom of the deposit page.
- Given the user taps/clicks Wire Transfer, when the instructions screen opens, then all required content is displayed: minimum amount ($10,000), sub-minimum return notice, Aeropay alternative note, name-match notice with the user's account name, Monday–Friday 4:00 PM EST deadline, location notice, Splash Sports bank details, the user's Splash Sports email as the Reference value, US banks only notice, and the closing instruction.
- Given the wire instructions screen is open, when the user taps/clicks the close/dismiss action, then they are returned to the deposit page.
- Given a user is not logged in or is located in a restricted state, when the deposit page loads, then the Wire Transfer option is not visible.
- Given the wire instructions screen is open, when the user views the account name field, then it displays the name on their Splash Sports account.
- Given the wire instructions screen is open, when the user views the Reference field, then it displays their Splash Sports account email.

## Dependencies

- **Bank account details** — Mock data is used during development. Production bank details (Receiving Bank Name, Receiving Bank Address, ABA/Routing Number, Account Number, Beneficiary Bank Account Title, Beneficiary Address) must be provided by finance/treasury and confirmed before launch.
- **Allowed state list** — Wire deposit visibility uses the same allowed-state gating as other deposit methods. Engineering must confirm which existing gate to hook into.
- **VP of Finance wire review process** — Incoming wires are manually reviewed and approved or denied by the VP of Finance. The ops workflow (how finance is notified of incoming wires, how approval/denial is communicated to the user) must be defined before launch.
- **Deposit page** — The existing deposit page at `https://app.splashsports.com/home` must support adding a Wire Transfer entry point at the bottom.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Production bank account details change and in-app instructions become stale | Medium | High | Store bank details in a config layer rather than hardcoded strings; define an ops process for updating them |
| Users initiate wires under $10,000 and are frustrated when funds are returned | Medium | Medium | Prominently display the $10,000 minimum on both the entry point and the instructions screen |
| VP of Finance wire review becomes a bottleneck at high volume | Low (v1) | Medium | Monitor wire volume post-launch; plan a more scalable approval workflow for v2 if needed |
| Location-gating misconfiguration exposes wire instructions to restricted-state users | Low | High | QA gate on location gating before launch; align with existing allowed-state logic used by other deposit methods |

## Open Questions

1. What are the exact production bank account details (routing number, account number, beneficiary info) and who owns keeping them current? _(Mock data used during development; finance/treasury to provide production values before launch.)_
2. How is the VP of Finance notified when a wire arrives for review? What is the communication flow back to the user when a wire is approved or denied? _(Ops workflow must be defined before launch.)_
3. Should the Wire Transfer option be surfaced in-app only, or also via email/push to users who may be interested?

## Sign-Off Log

| Role | Name | Status | Date | Notes |
|---|---|---|---|---|
| Product Lead | pfinnell | pending | — | — |
| Engineering Lead | — | pending | — | — |
| Design Lead | — | pending | — | — |
| QA Lead | — | pending | — | — |
| VP of Finance | — | pending | — | Wire approval/denial process must be confirmed |

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
