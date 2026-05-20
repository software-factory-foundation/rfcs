---
id: SFF-RFC-0005
title: Machine-Readable ADR Format (MR-ADR)
state: Discussion
working_group: WG-05
working_group_slug: architecture-and-rules
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Extends the conventional human-readable Architecture Decision Record with strict YAML front-matter and a small grammar for stating the decision, its forces, and its consequences in a form agents can query, link, and check for conflicts.

# Motivation

> TODO: working group to draft.

Classic ADRs are excellent for humans and useless for machines. As agents take on more architectural reasoning, they need ADRs that are queryable: what constraints are in force, what's been deprecated, what trade-offs were accepted.

# Detailed Design

> TODO: working group to draft.

## Proposed front-matter

- `id`, `title`, `status` (proposed, accepted, deprecated, superseded)
- `decision` — single normative statement
- `context[]` — forces and constraints driving the decision
- `consequences.positive[]`, `consequences.negative[]`
- `supersedes[]`, `superseded_by`
- `tags[]`

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- Michael Nygard ADR format (informal).
- MADR (Markdown Architectural Decision Records).
- arc42 templates.

# Prior Art

> TODO: working group to draft.

# Open Questions

- How are MR-ADR conflicts detected at scale?
- Should there be a registry of "Foundation-recognized" constraints?
