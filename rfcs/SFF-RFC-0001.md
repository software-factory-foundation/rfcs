---
id: SFF-RFC-0001
title: Portable Agent Memory Manifest v0
state: Discussion
working_group: WG-01
working_group_slug: memory-for-coding-agents
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines a portable, vendor-neutral manifest format that describes how a coding agent stores, retrieves, and exchanges memory across runtime boundaries. The goal is for an agent built on one platform to hand off its working context to an agent on a different platform without loss of meaning.

# Motivation

> TODO: working group to draft.

Today every agent platform invents its own memory representation. Handoff between platforms loses structure, attribution, and expiration semantics. A portable manifest creates the interoperability that the Foundation exists to enable.

# Detailed Design

> TODO: working group to draft. See [TEMPLATE.md](./TEMPLATE.md) for expected sections.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

> TODO: working group to draft.

# Prior Art

- Memento, MCP, Letta, and related agent memory frameworks.
- Vector store interchange formats.

# Open Questions

- How are credentials and PII tagged inside a memory manifest?
- What is the minimum schema every implementation MUST support?
- How does this compose with SFF-RFC-0014 (Context Expiration Policy Schema)?
