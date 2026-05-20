---
type: prd-one-pager
title: "User Exclusions & Limits Tab"
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

# User Exclusions & Limits Tab

> **Status:** `1pager-draft` · **Domain:** admin · **Product line:** internal-tool · **Author:** pfinnell

## Why now
When a user enrolls in self-exclusion today, the `self-exclusion` fraud flag is not applied automatically. T&S admin must notice the enrollment (via a Metabase query) and apply the flag manually. This gap leaves CS / Marketing unaware when a user enrolls in an exclusion and when the exclusion ends. Separately, self-imposed user limits (deposit limits, contest entry limits, and entry fee limits) are only visible via Metabase — there is no way for CS to see them.

## Problem
Self-exclusion enrollment and fraud flag enforcement are disconnected. A user can enroll in self-exclusion and CS has no insight to the enrollment. Similarly, when a self-exclusion expires, the flag must be manually removed — even though the expiry is time-determined and known to the system. Neither action requires human judgment, yet both require human intervention today.

User-set responsible gaming limits have the same blind spot: CS cannot see what limits a user has configured without running a Metabase query, making it impossible to quickly answer support inquiries or investigate disputes involving limits.

## Who
Trust & Safety team currently use two Metabase queries — `user-timeouts-self-exclusion` and `user-limits` — to look up per-user exclusion and limit enrollment. T&S can only apply a self-exclusion flag to the account for CS to see, but there's no insight into user limits.

## Solution sketch

### 1 — Backend: automate fraud flag lifecycle (identity service)

**On enrollment:** After the self-exclusion record is persisted, apply the `self-exclusion` fraud flag if not already present. Use `performedBy: 'system'` and note `"Auto-applied: user enrolled in self-exclusion"` so the action is visible in the admin audit history.

**On expiry:** When a self-exclusion expires (event-driven preferred; scheduled job as fallback), remove the `self-exclusion` fraud flag if currently applied. Use `performedBy: 'system'` and note `"Auto-removed: self-exclusion period expired"`.

Key decisions:
- Idempotent: if the flag is already `self-exclusion`, skip silently
- Flag failure does not roll back enrollment — user's intent is honored; ops alerted via Datadog
- If user has a different active fraud flag at enrollment, it is overwritten by `self-exclusion` (user-initiated and regulatory)
- Manual removal by an admin during an active exclusion will be overwritten on the next enrollment event

### 2 — Frontend: new Exclusions / Limits tab (admin tool)

Add an **Exclusions / Limits** tab to the user detail page in Splash Admin, surfacing data currently only available in Metabase.

**Exclusions section** — self-exclusion/timeout history for the user:
- Active exclusion indicator (dates, duration, status)
- Historical exclusions in reverse-chronological order

**Limits section** — responsible gaming limits the user has set:
- Daily Quick Picks spend limit (props service: `GET /daily_limits/{userId}`)
- Deposit limit override, if one exists (payments service: `GET /location-deposit-limit-overrides/{userId}`)
- Read-only display; limits are user-set and should not be editable by admins from this view

## Success signals
- The Metabase queries `user-timeouts-self-exclusion` and `user-limits` are no longer used operationally for per-user lookups — all data is visible directly in Admin
- Enrollment-to-flag lag drops from hours/days to near-zero
- CS can answer limit and exclusion inquiries without leaving the admin tool

## Risks & unknowns
- The exact self-exclusion endpoint and expiry mechanism in the identity service must be confirmed before implementation (event-driven vs. scheduled job changes the removal approach)
- The shape of the user-limits API response needs to be confirmed — it is unclear whether deposit limits and daily spend limits come from one endpoint or must be stitched from two services
- Admin-editability of limits is out of scope for now but may be requested as a follow-on

## Out of scope
- Admin ability to set or modify user-defined limits from the Exclusions / Limits tab
- Any change to what happens when an admin manually overrides the fraud flag during an active exclusion period
