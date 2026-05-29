---
type: prd-one-pager
title: "Admin User Transaction Visibility"
slug: admin-user-transaction-visibility
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
  - customer-support
---

# Admin User Transaction Visibility

> **Status:** `1pager-draft` · **Domain:** admin · **Product line:** internal-tool · **Author:** pfinnell

## Why now
Customer Support and Finance operators investigate transaction-related tickets daily — deposits not credited, withdrawal delays, incorrect balances — but the transaction history view in Splash Admin is too hard to read. Operators can't quickly parse what happened at a glance, forcing slow manual interpretation or escalations to engineering. Better visual presentation directly reduces handling time on the most common support workflow.

## Problem
Transaction history data exists on the Splash Admin user detail page, but the presentation makes it difficult to use. Key fields have no visual hierarchy, transaction types are displayed as internal codes operators must memorize, and there is no visual way to distinguish one transaction category from another. Scanning a user's history to find all deposits, or all failed transactions, requires reading every row individually.

## Who
Customer Support operators and Finance operators who investigate user account issues, balance discrepancies, and payment disputes in Splash Admin. Transaction-related tickets are among the most frequent investigation workflows.

## Solution sketch
- Display each transaction row with a clear visual hierarchy: human-readable type label, amount, status badge, and timestamp visible at a glance.
- Color-code each transaction row by type with the following assignments: Deposit=Green, Entry=Blue, Slip Entry=Light Blue, Payout=Gold, Slip Payout=Yellow, Contest/Entry Canceled=Grey, Withdrawal=Red, Bonus=Teal (Splash Sports brand teal).
- Replace internal type codes and enum values with human-readable labels.
- Show a colored status badge (Completed, Pending, Failed, Reversed) per transaction.
- Support row expansion to reveal secondary fields (transaction ID, linked slip/contest ID).
- Maintain reverse-chronological ordering (newest first).
- Read-only — no write operations in this PRD.

## Success signals
Operators can identify the category and status of any transaction on a user's history page in under 5 seconds without reading every row. Engineering escalations for "what happened to this transaction" queries decrease.

| Metric | Baseline | Target |
|---|---|---|
| Engineering escalations for transaction lookups | TBD | 0 |
| Operator time to identify a transaction type/status | TBD | < 5 sec |

## Risks & unknowns
The transaction type and status enum set needs to be inventoried with engineering to confirm the complete set of labels and color assignments. The Admin design system's available color palette for row-level coding needs to be confirmed with Design before implementation.

## Out of scope
Deposit refunds (separate PRD). Transaction search, filtering, or export. Any write operations.
