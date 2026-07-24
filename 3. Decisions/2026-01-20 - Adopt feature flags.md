---
type: decision
status: accepted
tags:
  - decisions
  - platform
---
## Decision

Adopt a feature-flag service to gate new features and enable gradual rollouts.

## Why

The [[2026-01-15 - Onboarding Revamp]] needs to ship incrementally and be reversible without redeploys.

## Alternatives considered

- Environment-based config toggles (rejected: no per-user targeting).
- Build in-house (rejected: not core to the product).

## Decision Owner

[[Jordan]]

## Revisit Trigger

Revisit if flag count grows unmanageable or the vendor pricing changes materially.

## AI Recall

When feature-flagging comes up again, start from this decision and explain what has changed since it was made.
