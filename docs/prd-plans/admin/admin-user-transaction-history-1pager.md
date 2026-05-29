---
type: prd-one-pager
title: "Admin User Transaction History — Deposit Refunds"
slug: admin-user-transaction-history
template_version: 1.0.0
domain: admin
product_line: internal-tool
status: 1pager-draft
created: 2026-05-29
updated: 2026-05-29
author: pfinnell
status_history:
  - status: 1pager-draft
    date: 2026-05-29
review_passes: []
shelved_reason: null
shelved_date: null
tags:
  - admin
  - users
  - transactions
  - finance
  - customer-support
---

# Admin User Transaction History — Deposit Refunds

> **Status:** `1pager-draft` · **Domain:** admin · **Product line:** internal-tool · **Author:** pfinnell

## Why now
T&S and CS agents process deposit refunds through the Adjust Balance section of Admin, which requires navigating away from the user's transaction history, manually copying and pasting transaction IDs, and cross-referencing two screens to confirm the outcome. This is friction on a routine, high-frequency support action.

## Problem
There is no Refund action on the transaction tab. Operators must leave the transaction history, use Adjust Balance with a manual transaction ID lookup, and return to verify the result. There is no in-context visibility into which deposits have already been refunded.

## Who
Customer Support operators who handle deposit disputes and Finance operators who oversee refund accuracy. Deposit refunds are a recurring support workflow.

## Solution sketch
- Add a Refund button to deposit transaction rows on the Admin user detail page, visible only to authorized roles.
- Require an explicit confirmation dialog before processing — defaults to the full deposit amount.
- Include a partial refund checkbox in the dialog; when checked, an amount input appears for the operator to enter a custom amount.
- Reflect the updated transaction status (Refunded) immediately after a successful refund.
- Show a clear error message if the refund fails.

## Success signals
Engineering escalations for deposit refund processing drop to zero. Operators can process a deposit refund in under 2 minutes without leaving Admin.

| Metric | Baseline | Target |
|---|---|---|
| Operator time to process a deposit refund | TBD | < 2 min |
| Clear visibility into refunded deposits | None | Refunded status visible on transaction row |

## Risks & unknowns
The full downstream call chain for a refund (wallet, ledger, payment processor) needs an engineering spike before scoping. Eligibility rules (which deposits can be refunded) and role-based access control must be confirmed before implementation.

## Out of scope
Refunds on non-deposit transaction types. Manual balance adjustments. Transaction display improvements (see Admin User Transaction Visibility PRD).
