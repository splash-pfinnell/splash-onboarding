---
type: prd-one-pager
title: "Admin User Verification Attempts"
slug: admin-user-verification-attempts
template_version: 1.0.0
domain: admin
product_line: internal-tool
status: 1pager-promoted
created: 2026-05-13
updated: 2026-05-13
author: pfinnell
status_history:
  - status: 1pager-draft
    date: 2026-05-13
  - status: 1pager-ready
    date: 2026-05-13
    review_pass_index: 1
  - status: 1pager-promoted
    date: 2026-05-13
    linked_prd: admin/admin-user-verification-attempts.md
    body_hash_at_promotion: sha256:e28f73571f15bb0e21c82003e8ff86e1b98c03f1ef25ebd9850d2175c3bd0245
review_passes:
  - date: 2026-05-13
    product_score: 68
    findings:
      - severity: CRITICAL
        dimension: success-signals
        message: "Success signal reads as an acceptance criterion, not a measurable outcome. No metric, direction of change, or time-bound target named."
      - severity: ERROR
        dimension: target-user
        message: "Persona named correctly but no volume or frequency signal provided (how many operators, how often does this pain occur)."
    catalog_check_run: false
  - date: 2026-05-13
    product_score: 75
    findings:
      - severity: ERROR
        dimension: target-user
        message: "Frequency signal ('hundreds of lookups per day') lives in Success signals, not in Who. Persona lacks volume context in its own section."
    catalog_check_run: false
linked_prd: admin/admin-user-verification-attempts.md
shelved_reason: null
shelved_date: null
tags:
  - admin
  - users
  - verification
  - trust-and-safety
---

# Admin User Verification Attempts

> **Status:** `1pager-promoted` · **Domain:** admin · **Product line:** internal-tool · **Author:** pfinnell

## Why now
Support and Trust & Safety teams currently split their workflow across two tools — Splash Admin and Socure — just to investigate a single user's verification history. Consolidating verification attempt data into Admin reduces context-switching, speeds up investigations, and gives Trust & Safety visibility into suspicious attempt patterns they currently can't see until they manually look up the user in Socure.

## Problem
Splash Admin only surfaces a user's latest verification attempt and a pass/fail status — full attempt history and failure reasons are not visible. Operators cannot see all verification attempts, the data the user submitted, or why individual attempts failed without leaving Admin and querying Socure directly.

## Who
Splash Admin users on the Customer Support and Trust & Safety teams.

## Solution sketch
- On the Splash Admin user details page, add a Verification section showing all historical attempts.
- Each attempt displays the data submitted by the user and the reason for failure (where applicable).
- All attempts are shown in reverse-chronological order — not just the most recent.

## Success signals
Support and Trust & Safety teams currently perform hundreds of Socure lookups per day to investigate user verification attempts. Success means that number drops to zero — all verification history is available directly in Admin, eliminating the need to leave the tool.

## Risks & unknowns
The primary dependency is pulling verification attempt data from Socure (dashboard.socure.com/#/transactions). API availability, rate limits, and the shape of Socure's transaction data need to be confirmed with engineering before scoping the implementation.

## Out of scope
Users who have not attempted or completed verification. This feature only surfaces data for users with existing verification attempts in Socure.
