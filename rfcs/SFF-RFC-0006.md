---
id: SFF-RFC-0006
title: Universal Test Result Envelope (UTRE)
state: Pre-Draft
working_group: WG-06
working_group_slug: testing-and-test-integrations
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines a standard envelope for test results emitted by factory pipelines: identity, status, evidence, environment, lineage, and severity. UTRE is transport-neutral and accumulates across stages of a single change.

# Motivation

> TODO: working group to draft.

JUnit XML is a quarter-century old and was never designed for AI-augmented pipelines. Modern factories need to attach reasoning, generated test code, screenshots, agent traces, and environment provenance to results — without inventing a new envelope per tool.

# Detailed Design

> TODO: working group to draft.

## Proposed envelope fields

- `id` — globally unique result ID
- `change_id` — identifier of the change under test
- `stage` — pipeline stage (unit, integration, e2e, …)
- `status` — pass, fail, flake, skipped, error
- `severity` — informational, minor, major, blocker
- `evidence[]` — links to logs, screenshots, traces, artifacts
- `environment` — OS, runtime, deps
- `lineage` — what produced this result (tool, agent, version)
- `findings[]` — structured assertions (composable with SFF-RFC-0008 SROS)

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- JUnit XML, TAP, xunit-viewer formats.
- OpenTelemetry test signal (still draft).

# Prior Art

> TODO: working group to draft.

# Open Questions

- Streaming vs batch — when should a UTRE be considered "final"?
- How does UTRE compose with SFF-RFC-0008 (Structured Review Output Schema)?
