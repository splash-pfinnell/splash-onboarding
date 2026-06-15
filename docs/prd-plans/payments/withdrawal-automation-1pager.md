---
type: prd-one-pager
title: "Withdrawal Automation"
slug: withdrawal-automation
template_version: 1.0.0
domain: payments
product_line: admin
status: 1pager-draft
created: 2026-06-09
updated: 2026-06-09
author: pfinnell
status_history:
  - status: 1pager-draft
    date: 2026-06-09
review_passes: []
linked_prd: payments/withdrawal-automation.md
shelved_reason: null
shelved_date: null
tags:
  - payments
  - withdrawals
  - automation
  - admin
---

# Withdrawal Automation

> **Status:** `1pager-draft` · **Domain:** payments · **Product line:** admin · **Author:** pfinnell

## Why now
Withdrawal processing is currently handled with fixed automation rules that do not account for dollar amount or the need for emergency intervention. As withdrawal volume grows, ops needs per-method controls — both threshold-based automation and a pause lever — to manage risk and comply with evolving compliance requirements without code deploys.

## Problem
Admins have no in-product way to adjust how withdrawals are automatically processed. When a payment method has a risk event or a compliance hold is needed, the only option is an engineering change. There is also no way to differentiate automation behavior by dollar amount (e.g., auto-approve small withdrawals, manually review large ones) on a per-method basis.

## Who
Internal admins (Finance, Compliance, Risk) with access to the Payments admin page. End users are not aware of these controls.

## Solution sketch
Add a **Withdrawal Automation** tab to the Payments admin page with a row per payment method: **Aeropay, Card, PayPal, Skrill, Venmo**. Each row exposes:

- **Threshold control** — a per-method dollar amount input. A withdrawal is auto-processed when it is at or below the method's threshold AND the user has only one payment method linked to their account. Withdrawals that do not meet both conditions are routed to the existing manual withdrawal processing queue. V1 launch thresholds:

  | Method  | Threshold |
  |---------|-----------|
  | Card    | $250      |
  | PayPal  | $500      |
  | Venmo   | $500      |
  | Aeropay | $1,000    |
  | Skrill  | No automation (manual queue only) |
- **Pause toggle** — a binary on/off switch that immediately halts automated processing of new withdrawal requests for that method. Withdrawals already in a `Processing` state are unaffected and continue to completion. Paused methods display a visible status indicator so the current state is unambiguous at a glance.
- **Last modified** — the timestamp and admin username of the most recent change per method, for audit purposes.

**Access control:**
- Super Admin — full read/write access to all automation settings.
- Trust & Safety — read-only view of all settings; no ability to modify.

All changes require a confirmation dialog before taking effect. Changes apply immediately to new withdrawal requests with no deploy required. An in-UI audit trail displays the last 12 months of changes, visible to both Super Admin and Trust & Safety. If a processing error occurs after a setting change, Super Admins are notified via the admin UI and Slack.

## Success signals
- Finance/Compliance can pause a payment method and adjust thresholds within the admin UI in under 2 minutes, with no eng involvement.
- Zero incidents where automation continued to run after an intended pause.
- Audit log is used in at least one compliance or risk review within 90 days of launch.

## Risks & unknowns
- The existing manual withdrawal processing queue must be able to absorb increased volume if automation is paused across multiple methods simultaneously.
- Slack channel and webhook for processing error alerts TBD — engineering to confirm existing payments notification integration.

## Out of scope
- Per-user or per-geography automation rules — v1 is method-level only.
- Scheduling automation pauses in advance (e.g., "pause every Friday at 5 PM") — v1 is manual toggle only.
- Surfacing automation state to end users or in the user-facing withdrawal flow.
