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
Customer Support operators resolving deposit disputes have no way to initiate a refund from Splash Admin. Every refund requires escalating to engineering or switching to a separate payment tool, adding unnecessary handling time and engineering load for a routine, high-frequency support action.

## Problem
There is no Refund action on the Admin user detail page. Operators who need to reverse a deposit must leave Admin, engage engineering, and manually verify the result — a multi-step, multi-person process for what should be a single operator action.

## Who
Customer Support operators who handle deposit disputes and Finance operators who oversee refund accuracy. Deposit refunds are a recurring support workflow.

## Solution sketch
- Add a Refund button to deposit transaction rows on the Admin user detail page, visible only to authorized roles.
- Require an explicit confirmation step before processing.
- Reflect the updated transaction status (Reversed) immediately after a successful refund.
- Show a clear error message if the refund fails.

## Success signals
Engineering escalations for deposit refund processing drop to zero. Operators can process a deposit refund in under 2 minutes without leaving Admin.

| Metric | Baseline | Target |
|---|---|---|
| Engineering escalations for deposit refunds | TBD | 0 |
| Operator time to process a deposit refund | TBD | < 2 min |

## Risks & unknowns
The full downstream call chain for a refund (wallet, ledger, payment processor) needs an engineering spike before scoping. Eligibility rules (which deposits can be refunded) and role-based access control must be confirmed before implementation.

## Out of scope
Refunds on non-deposit transaction types. Partial refunds. Manual balance adjustments. Transaction display improvements (see Admin User Transaction Visibility PRD).
