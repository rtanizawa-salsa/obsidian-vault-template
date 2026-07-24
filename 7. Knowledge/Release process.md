---
type: reference
tags:
  - knowledge
  - platform
---
# Release process

Durable reference for how changes ship to production. (Fictional example content.)

## Flow

1. Open a pull request; CI runs tests and lint.
2. At least one review approval required to merge.
3. Merges to `main` deploy to staging automatically.
4. Promote to production via the deploy dashboard once staging checks pass.

## Feature flags

New user-facing behavior ships behind a flag per [[2026-01-20 - Adopt feature flags]]. Roll out gradually, then remove the flag once stable.

## Rollback

Re-promote the previous release from the deploy dashboard; disable the offending flag first if the change was flag-gated.
