---
id: SFF-RFC-0008
title: Structured Review Output Schema (SROS)
state: Discussion
working_group: WG-08
working_group_slug: code-review-processes
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Specifies the output schema for an automated or human-augmented code review: findings, severity, anchor (file/line/symbol), category, suggested fix, and disposition. Multiple SROS outputs from different reviewers can be merged into a single review.

# Motivation

> TODO: working group to draft.

Code review tooling speaks a hundred different output dialects: GitHub PR comments, SonarQube reports, Semgrep findings, agent-written prose. Merging them into a coherent reviewer experience requires a common schema so the front-end can deduplicate, group by file, and surface the highest-severity findings.

# Detailed Design

> TODO: working group to draft.

## Proposed finding shape

- `finding_id`, `reviewer_id`, `reviewer_kind` (human, llm, linter, …)
- `anchor` — file, line range, optional symbol
- `category` — bug, smell, style, security, performance, doc
- `severity` — informational, minor, major, blocker
- `message` — natural-language explanation
- `suggested_fix` — optional patch
- `disposition` — open, accepted, declined, deferred
- `evidence[]` — links to traces, examples, references

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- SARIF (Static Analysis Results Interchange Format).
- GitHub's PR review API shape.

# Prior Art

> TODO: working group to draft.

# Open Questions

- How does SROS compose with SARIF for legacy tools?
- What is the minimum interoperability set every implementer MUST support?
