---
id: SFF-RFC-0019
title: OpenLineage Compatibility Layer
state: Pre-Draft
working_group: WG-02
working_group_slug: metadata-storage-for-factories
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines a mapping between Factory Manifest events (SFF-RFC-0002) and the [OpenLineage](https://openlineage.io) specification so factories can publish to existing OpenLineage consumers without parallel instrumentation.

# Motivation

> TODO: working group to draft.

OpenLineage is the de facto data-lineage event spec. Factories that emit Foundation manifest events should be able to also satisfy OpenLineage consumers (Marquez, Datakin, vendor backends) by emitting a compatible projection — without re-instrumenting their pipeline.

# Detailed Design

> TODO: working group to draft.

## Mapping outline

- Factory `run` events → OpenLineage `RunEvent`
- Factory artifacts → OpenLineage `Dataset`
- Factory jobs / agents → OpenLineage `Job`
- Field-level translation tables included as an appendix

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- Replace OpenLineage in the SFF ecosystem entirely (rejected — it already has wide adoption).
- Native OpenLineage emission only (rejected — loses factory-specific fields).

# Prior Art

- OpenLineage specification.

# Open Questions

- Are bidirectional consumers (read OpenLineage → produce SFF Factory Manifest events) in scope for v0?
- Versioning policy — track OpenLineage releases or pin?
