---
id: SFF-RFC-0002
title: Factory Manifest v1 — Core Fields
state: Discussion
working_group: WG-02
working_group_slug: metadata-storage-for-factories
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Specifies the minimum set of core fields that every software factory MUST publish to describe itself: identity, capabilities, supported agent types, telemetry endpoints, and conformance claims. The Factory Manifest is the document an external observer reads to understand what a factory is and what it does.

# Motivation

> TODO: working group to draft.

Cross-factory orchestration, observability, and conformance testing all require a single document that names the factory and enumerates its capabilities. Without a standard core, every integration is bespoke.

# Detailed Design

> TODO: working group to draft.

## Core Fields (proposed)

- `id` — globally unique identifier
- `version` — factory manifest schema version
- `name`, `vendor`, `homepage`
- `capabilities[]` — declared capabilities
- `agents[]` — agent types this factory hosts
- `telemetry` — endpoints and emitted event types
- `conformance[]` — Foundation RFCs the factory claims conformance to

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

> TODO: working group to draft.

# Prior Art

- Backstage `catalog-info.yaml`.
- OpenAPI / AsyncAPI service descriptions.
- The OCI Image Spec manifest.

# Open Questions

- Where does the manifest live? Repo root, served from an endpoint, or both?
- How is the manifest signed?
- What is the relationship to SFF-RFC-0019 (OpenLineage Compatibility Layer)?
