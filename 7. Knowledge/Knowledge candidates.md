---
type: knowledge
tags:
  - knowledge
  - inbox
---
# Knowledge candidates (inbox)

Durable facts vault-groom distilled from daily notes, pending promotion into `7. Knowledge/`. Review, then move each into a real Knowledge note and delete the line.

Run `/promote-knowledge` to have Claude group these by subject and either enrich the existing Knowledge note on that topic or create a new one at the right altitude. (Fictional example candidates below — delete them when you start using the vault.)

## 2026-01-22

- [ ] New user-facing behavior ships behind a feature flag, rolled out gradually, with the flag removed once the change is stable. Rollback disables the flag before re-promoting the previous release.
	- **Update:** [[Release process]] — add to the existing "Feature flags" section
	- Source: [[2026-01-20 - Adopt feature flags]]
	- Confidence: high
- [ ] Feature flags were chosen over environment-based config toggles because config toggles have no per-user targeting; building in-house was rejected as not core to the product. Revisit if flag count grows unmanageable or vendor pricing changes materially.
	- **New note:** `Feature flag conventions` (area: Engineering)
	- Source: [[2026-01-20 - Adopt feature flags]]
	- Confidence: medium
- [ ] The team prefers async design docs circulated before implementation starts.
	- **New note:** `Team working agreements` (area: Process)
	- Source: [[2026-01-22 - Weekly team sync]]
	- Confidence: medium
