---
id: SFF-RFC-0003
title: Agent-Readable Issue Format (ARIF)
state: Discussion
working_group: WG-03
working_group_slug: planning-and-issue-management
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines a structured issue representation that both humans and AI agents can read, write, and decompose. ARIF supersedes the informal Markdown body of typical issues with explicit fields for dependencies, success criteria, agent ownership, and decomposition state.

# Motivation

> TODO: working group to draft.

When agents plan and execute work, the unit of work has to be machine-readable: with explicit dependencies, acceptance criteria, and ownership. Treating issues as unstructured Markdown forces agents to re-derive structure on every read, expensively and inconsistently.

# Detailed Design

> TODO: working group to draft.

## Proposed core fields

- `id`, `title`, `state`
- `description` — human-authored summary
- `acceptance_criteria[]` — testable conditions
- `depends_on[]`, `blocks[]`
- `assignee` — human, agent, or unassigned
- `decomposition` — list of child issue IDs, or "not yet decomposed"
- `provenance` — who/what created this issue

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- GitHub Issues form schemas.
- Linear's structured fields.
- Beads (the Foundation's own dependency-aware issue tracker).

# Prior Art

> TODO: working group to draft.

# Open Questions

- Does ARIF live alongside an existing tracker (GitHub, Linear, Jira) as metadata, or is it standalone?
- How is decomposition tracked across nested agents?
