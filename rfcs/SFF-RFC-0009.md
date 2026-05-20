---
id: SFF-RFC-0009
title: Factory Deployment Manifest v0
state: Pre-Draft
working_group: WG-09
working_group_slug: code-deployment
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines the manifest a software factory emits to a deployment system to release a build: identity, environment targets, approval signatures, rollback criteria, and verification probes. Manifests are signed and replayable.

# Motivation

> TODO: working group to draft.

When a factory produces a deployable artifact, the deploy system needs more than the artifact — it needs the policy decisions that authorize the deploy, the environments it's targeted at, and the conditions under which it should be rolled back. A standard manifest lets any factory deploy through any compliant deployer.

# Detailed Design

> TODO: working group to draft.

## Proposed manifest fields

- `id`, `artifact_ref`, `version`
- `target_environments[]`
- `approvals[]` — signed approval records
- `rollback_criteria` — observable signals that trigger rollback
- `verification_probes[]` — post-deploy health checks
- `provenance` — link to the change and the factory that produced it

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- SLSA (Supply-chain Levels for Software Artifacts).
- in-toto attestations.

# Prior Art

> TODO: working group to draft.

# Open Questions

- Signature scheme (Sigstore / GPG / custom)?
- How does this interoperate with existing GitOps tools (Argo, Flux)?
