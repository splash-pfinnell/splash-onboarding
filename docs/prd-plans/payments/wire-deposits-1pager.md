---
type: prd-one-pager
title: "Wire Deposits"
slug: wire-deposits
template_version: 1.0.0
domain: payments
product_line: consumer
status: 1pager-draft
created: 2026-05-14
updated: 2026-05-14
author: pfinnell
status_history:
  - status: 1pager-draft
    date: 2026-05-14
review_passes: []
linked_prd: null
shelved_reason: null
shelved_date: null
tags:
  - payments
  - deposits
  - wire-transfer
  - vip
---

# Wire Deposits

> **Status:** `1pager-draft` · **Domain:** payments · **Product line:** consumer · **Author:** pfinnell

## Why now
Users who want to make large deposits ($10,000+) have no in-app pathway to do so — wire transfer is the standard method for moving large sums and is expected by serious players. Adding a wire option to the deposit page removes a gap in the product and eliminates support contacts from users asking how to make large deposits.

## Problem
The Splash Sports deposit page has no wire transfer option. Users who want to deposit $10,000 or more have no in-app way to do so — they must contact support or use workarounds. This creates friction and limits Splash's ability to grow large-deposit volume.

## Who
All logged-in Splash Sports users located in an allowed state. The Wire Transfer option appears at the bottom of the deposit page for all eligible users.

## Solution sketch
- Add a Wire Transfer option at the bottom of the deposit page, visible to all logged-in users in allowed states.
- When tapped/clicked, display a read-only wire instructions screen with:
  - The $10,000 minimum requirement and a note that sub-minimum wires will be returned; Aeropay recommended for smaller deposits
  - A reminder that the user's bank account name must match their Splash Sports account name (shown dynamically)
  - Splash Sports bank account details (Receiving Bank Name, Receiving Bank Address, ABA/Routing Number, Account Number, Beneficiary Bank Account Title, Beneficiary Address)
  - The user's Splash Sports account email as the Reference field value (shown dynamically)
  - Deadline: wires must arrive Monday–Friday by 4:00 PM EST to post same day
  - US banks only; location notice that user must be in a valid state for credit to post
- A close/dismiss action returns the user to the deposit page — no further in-app action required.
- Incoming wires are reviewed and approved or denied by the VP of Finance.

## Success signals
Wire deposit volume (count and dollar amount) grows month-over-month after launch. Support contacts related to "how do I make a large deposit" drop to near zero as users find the in-app instructions.

## Risks & unknowns
- Production bank account details must be provided by finance/treasury before launch; mock data is used during development.
- The VP of Finance wire review/approval workflow (notification, approval, user communication) must be defined before launch.
- Location gating must align with the existing allowed-state logic used for other deposit methods.

## Out of scope
- Automated wire matching or reconciliation (v1 relies on the Reference/email field and VP of Finance manual review).
- International banks — US banks only in v1.
- Initiating or canceling wires from within the app — users initiate wires through their own bank.
