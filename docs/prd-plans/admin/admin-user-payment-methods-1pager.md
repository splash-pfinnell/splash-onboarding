---
type: prd-one-pager
title: "Admin User Payment Methods"
slug: admin-user-payment-methods
template_version: 1.0.0
domain: admin
product_line: internal-tool
status: 1pager-draft
created: 2026-08-27
updated: 2026-08-27
author: pfinnell
status_history:
  - status: 1pager-draft
    date: 2026-08-27
review_passes: []
shelved_reason: null
shelved_date: null
tags:
  - admin
  - users
  - payments
  - customer-support
  - trust-and-safety
---

# Admin User Payment Methods

> **Status:** `1pager-draft` · **Domain:** admin · **Product line:** internal-tool · **Author:** pfinnell

## Why now
CS, T&S, and Finance operators have no way to see or act on which payment methods are linked to a user's account from within Splash Admin. Operators either escalate to Payments engineering or try to infer method state from transaction history line items. When a specific method needs to be removed — a chargeback-prone card, a compromised account, a method a user wants gone — operators today have no in-product lever and must file an engineering request.

## Problem
There is no Payment Methods view on the Admin user detail page, and no way to remove a linked method. Operators cannot see which methods (Aeropay, Card, PayPal, Venmo, Skrill) are linked to an account, when they were added or last used, and cannot remove a method entirely without an engineering ticket.

## Who
Customer Support and Trust & Safety operators handling account and payment investigations; Finance operators assessing account risk. This is a recurring need during support tickets and fraud/dispute reviews.

## Solution sketch
- Add a "Payment Methods" tab to the Admin user detail page.
- List each linked method: type, masked identifier (last 4 only), status, date added, last used.
- Let an authorized operator remove a payment method from the account, with confirmation before the removal applies.
- Removed/deleted methods stay out of the default view, hidden behind a "Show removed methods" toggle, for historical context.
- This is a standalone control surface for payment method removal — it does not read from or write to withdrawal automation's thresholds, pause state, or eligibility logic.

## Success signals
Escalations to Payments engineering for payment-method lookups and removal requests drop to near zero. Operators can look up or remove a method in under a minute inside Admin instead of filing a ticket.

## Risks & unknowns
Needs confirmation from Payments engineering on whether a single API supports both reading all linked methods and writing a removal per method, and how quickly a removal takes effect downstream (deposit/withdrawal flows).

## Out of scope
Restricting a method to deposits only or withdrawals only, or any per-method usage-mode setting. Unmasking full card/account numbers. Adding a new payment method from Admin. Restoring a removed/deleted method. Any coupling to withdrawal automation's per-method thresholds or pause controls — those remain owned by the Withdrawal Automation PRD and are a separate system.
