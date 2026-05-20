---
type: prd-one-pager
title: "Auto-Apply Self-Exclusion Fraud Flag"
slug: self-exclusion-fraud-flag
template_version: 1.0.0
domain: admin
product_line: internal-tool
status: 1pager-draft
created: 2026-05-20
updated: 2026-05-20
author: pfinnell
status_history:
  - status: 1pager-draft
    date: 2026-05-20
tags:
  - admin
  - users
  - self-exclusion
  - trust-and-safety
  - responsible-gaming
  - identity-service
---

# Auto-Apply Self-Exclusion Fraud Flag

> **Status:** `1pager-draft` · **Domain:** admin · **Product line:** internal-tool · **Author:** pfinnell

## Why now
When a user enrolls in self-exclusion today, the `self-exclusion` fraud flag — which enforces Deny Contest and Deny Deposit — is not applied automatically. An admin must notice the enrollment (via a Metabase query) and apply the flag manually. This gap leaves self-excluded users unrestricted until someone catches it, which is both a compliance risk and an operational burden.

## Problem
Self-exclusion enrollment and fraud flag enforcement are disconnected. A user can enroll in self-exclusion and still deposit or enter contests until an admin manually applies the flag. Similarly, when a self-exclusion expires, the flag must be manually removed — even though the expiry is time-determined and known to the system. Neither action requires human judgment, yet both require human intervention today.

## Who
Trust & Safety and Internal Ops teams who currently monitor a Metabase query (`user-timeouts-self-exclusion`) to identify self-excluded users who need the fraud flag applied.

## Solution sketch
In the identity service, hook into the self-exclusion enrollment and expiry lifecycle to automate fraud flag management:

**On enrollment:** After the self-exclusion record is persisted, apply the `self-exclusion` fraud flag if not already present. Use `performedBy: 'system'` and note `"Auto-applied: user enrolled in self-exclusion"` so the action is visible in the admin audit history.

**On expiry:** When a self-exclusion expires (event-driven or via scheduled job), remove the `self-exclusion` fraud flag if it is currently applied. Use `performedBy: 'system'` and note `"Auto-removed: self-exclusion period expired"`.

**Key decisions:**
- Idempotent: if the flag is already `self-exclusion`, skip silently
- Flag failure does not roll back enrollment — user's self-exclusion intent is honored; ops alerted via Datadog
- If user has a different active fraud flag at enrollment time, it is overwritten by `self-exclusion` (user-initiated and regulatory; documented in PR)
- Manual removal by an admin during an active self-exclusion will be overwritten on the next enrollment event (document this behavior)

No admin tool changes are required. The existing fraud flag badge, restrictions display, and audit history tab in Splash Admin surface the automated changes immediately.

## Success signals
The Metabase query `user-timeouts-self-exclusion` is no longer used operationally — every self-excluded user has the fraud flag applied within seconds of enrollment, with no manual admin action. Enrollment-to-flag lag drops from hours/days to near-zero.

## Risks & unknowns
- The exact self-exclusion endpoint and expiry mechanism in the identity service must be confirmed before implementation (event-driven expiry vs. scheduled job changes the removal approach)
- If expiry is time-based with no explicit hook, a scheduled job introduces polling delay — this should be acceptable but needs to be agreed on

## Out of scope
- Surfacing self-exclusion enrollment dates or duration in the admin UI (separate ticket)
- Any change to what happens when an admin manually overrides the fraud flag during an active exclusion period
