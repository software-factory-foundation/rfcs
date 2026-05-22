---
id: SFF-RFC-0002
title: Factory Manifest v1 — Core Fields
state: Discussion
working_group: WG-02
working_group_slug: metadata-storage-for-factories
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Specifies the minimum set of core fields that every software factory MUST publish to describe itself: identity, capabilities, supported agent types, telemetry endpoints, and conformance claims. The Factory Manifest is the document an external observer reads to understand what a factory is and what it does.

# Motivation

> TODO: working group to draft.

Cross-factory orchestration, observability, and conformance testing all require a single document that names the factory and enumerates its capabilities. Without a standard core, every integration is bespoke.

## Related Work

Three adjacent manifest traditions inform the Factory Manifest, and no single one is sufficient on its own.

**Supply-chain identity manifests** establish the *who and what* of software artefacts. [SPDX 3.0](https://spdx.github.io/spdx-spec/v3.0.1/) uses globally-resolvable `spdxId` URIs and a graph-native element model; [CycloneDX 1.6](https://cyclonedx.org/specification/overview/) adds first-class `services` and `formulation` (build pipeline) sections alongside components, taking it closer to a factory's surface than SPDX; [SLSA Provenance v1.0](https://slsa.dev/spec/v1.0/provenance) makes the *identity of the builder* a first-class signed claim (`builder.id`); [OpenSSF Scorecard](https://github.com/ossf/scorecard) expresses conformance as numeric per-check scores rather than boolean compliance; [GUAC](https://guac.sh/) demonstrates that identity-resolution *across* heterogeneous attestations is itself a useful service.

**Container and orchestration manifests** establish the *declarative shape* used by Kubernetes-era tooling. The [OCI Image Manifest](https://github.com/opencontainers/image-spec/blob/main/manifest.md) is content-addressed by digest; the [OCI Artifact Manifest](https://oras.land/docs/concepts/artifact/) lets a single `artifactType` string declare *what kind of thing* the manifest describes (pluggable taxonomy via IANA media types); the [Kubernetes CRD pattern](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) splits `spec` (author intent) from `status` (controller-observed) — a separation absent from supply-chain manifests; [Helm Chart.yaml](https://helm.sh/docs/topics/charts/) usefully splits *artefact version* from *contained-application version*; [OpenAPI 3.1](https://spec.openapis.org/oas/v3.1.0) aligns its `info.license.identifier` field with SPDX expressions, demonstrating cross-tradition reuse.

**Developer-experience catalogs** establish *which fields humans maintain by hand vs. which are auto-discovered*. [Backstage](https://backstage.io/docs/features/software-catalog/descriptor-format/) (Apache-2.0) makes `spec.owner` a *required reference* to a Group / User — ownership is structural, not advisory. [Roadie](https://roadie.io/) and [Cortex](https://docs.cortex.io/ingesting-data-into-cortex/entities/yaml) (proprietary) extend the pattern: Cortex's globally-unique `x-cortex-tag` is a notable precedent for cross-organization factory identity; [OpsLevel](https://docs.opslevel.com/docs/opslevel-yml)'s stable *aliases* for tier / lifecycle keep enums referenceable rather than free-text.

The gap SFF-RFC-0002 fills sits between these traditions. None models **supported AI agents** or **agent capability declarations** as first-class fields — CycloneDX's AI-BOM components describe ML models, not the agents that drive a factory. The manifest must invent novel fields here (e.g. `supportedAgents[]`) while reusing CycloneDX's signed metadata block, SLSA's `builder.id` pattern, the Kubernetes envelope shape, OCI's `artifactType` taxonomy, Backstage's required-reference ownership, and Cortex's globally-unique identity.

# Detailed Design

> TODO: working group to draft.

## Core Fields (informed by prior art)

The manifest SHOULD adopt the Kubernetes-style declarative envelope and the supply-chain signed-metadata block.

**Envelope (Kubernetes CRD-shaped):**

- `apiVersion: sff.dev/v1`
- `kind: FactoryManifest`
- `metadata` — name, namespace, labels, annotations, uid.
- `spec` — author intent (everything below).
- `status` — observed runtime state (populated by the factory).

**Identity and provenance (CycloneDX + SLSA):**

- `id` — globally unique identifier (URN or URI, mirrors Cortex `x-cortex-tag`).
- `name`, `version` — artefact version and `appVersion` for the contained pipeline (Helm split).
- `vendor`, `homepage` — supplier metadata (CycloneDX `metadata.supplier`).
- `builder.id` — signed identity of the publisher (SLSA Provenance v1.0 pattern).
- `serialNumber` — UUID URN (CycloneDX) for one-shot snapshot identity.
- `artifactType` — IANA-style media type (OCI) declaring the factory class.

**Capabilities, agents, conformance (new in MR-RFC-0002):**

- `capabilities[]` — declared capabilities, each a `{ id, version, attestation? }` tuple.
- `supportedAgents[]` — first-class agent declarations: `{ model, provider, version_range, mcp_surface? }`. **No surveyed tradition has this**; this is the manifest's novel contribution.
- `conformance[]` — numeric per-RFC conformance scores (OpenSSF Scorecard pattern: each entry is `{ rfcId, score: 0..10, reason?, evidence_url? }`, not boolean).
- `telemetry` — endpoints and emitted event types; composes with SFF-RFC-0019 (OpenLineage Compatibility Layer).

**Ownership and lifecycle (Backstage + OpsLevel):**

- `owner` — required reference to a Group / User entity (not free text).
- `lifecycle` — stable alias from a published enum (`experimental` | `production` | `deprecated`).
- `tier` — stable alias from a published enum.

## Discoverability

A factory SHOULD publish its manifest at `/.well-known/sff-factory-manifest.json` (matching the OpsLevel / Backstage convention of a fixed filename for auto-discovery) and SHOULD also expose it as an OCI Artifact (`artifactType: application/vnd.sff.factory-manifest.v1+json`) so registries index it.

## Signing

The manifest SHOULD be wrapped in a DSSE envelope (in-toto / Sigstore pattern) so identity claims are cryptographically verifiable. The verifier anchors on `builder.id` (SLSA pattern).

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[Backstage `catalog-info.yaml`](https://backstage.io/docs/features/software-catalog/descriptor-format/)** (Apache-2.0). Closest existing developer-catalog precedent; the v1 manifest adopts its envelope and required-reference ownership.
- **[OpenAPI / AsyncAPI service descriptions](https://spec.openapis.org/oas/v3.1.0)** (Apache-2.0). Strong for HTTP API surfaces but doesn't model AI-agent capabilities or conformance claims.
- **[OCI Image Manifest](https://github.com/opencontainers/image-spec/blob/main/manifest.md)** (Apache-2.0). Provides the `artifactType` taxonomy pattern adopted here; not a factory schema by itself.
- **[CycloneDX 1.6](https://cyclonedx.org/specification/overview/)** (Apache-2.0). Closest supply-chain precedent. Borrow `metadata` (supplier / manufacturer / tools / timestamp) and `services` / `formulation` patterns.
- **[SLSA Provenance v1.0](https://slsa.dev/spec/v1.0/provenance)** (CC-BY-4.0). Borrow the `builder.id` signed-identity claim.
- **[OpenSSF Scorecard](https://github.com/ossf/scorecard)** (Apache-2.0). Numeric conformance scoring pattern.
- **[Cortex YAML](https://docs.cortex.io/ingesting-data-into-cortex/entities/yaml)** (proprietary). Globally-unique tag requirement.
- **[OpsLevel `opslevel.yml`](https://docs.opslevel.com/docs/opslevel-yml)** (proprietary). Stable alias pattern for tier / lifecycle enums.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Identity field | Capability / conformance fields | Auto-discoverable? | Contribution to v1 |
|---|---|---|---|---|---|
| [SPDX 3.0](https://spdx.github.io/spdx-spec/v3.0.1/) | Supply-chain | `spdxId` URI | License conclusions, relationships | Partial (tool-generated) | Graph-native, URI identity |
| [CycloneDX 1.6](https://cyclonedx.org/specification/overview/) | Supply-chain | `serialNumber` UUID URN | `services`, `formulation`, vulns, VEX | Yes (CI-generated) | `metadata` block, signing patterns |
| [SLSA Provenance v1.0](https://slsa.dev/spec/v1.0/provenance) | Supply-chain | `builder.id` + `subject` digest | `buildType`, `resolvedDependencies` | Yes (CI-generated) | Signed `builder.id` claim |
| [OpenSSF Scorecard](https://github.com/ossf/scorecard) | Supply-chain | `repo.name/commit` | Numeric per-check scores | Yes (scanner) | Conformance as scores, not booleans |
| [GUAC / Trustify](https://guac.sh/) | Supply-chain | Normalized graph IDs | CertifyGood/Bad, HasSLSA, CertifyVuln | Yes (ingest) | Identity resolution as a service |
| [OCI Image Manifest](https://github.com/opencontainers/image-spec/blob/main/manifest.md) | Container | Content digest | `annotations`, `artifactType`, `subject` | Yes (registry) | Manifest is its own ID |
| [OCI Artifact Manifest](https://oras.land/docs/concepts/artifact/) | Container | Digest + `artifactType` | `artifactType` taxonomy | Yes (registry referrers) | Pluggable factory-class taxonomy |
| [Kubernetes CRD](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) | Orchestration | `name`+`namespace`+`uid` | `spec` / `status` separation | Yes (API server) | Envelope, intent-vs-observed split |
| [Helm Chart.yaml](https://helm.sh/docs/topics/charts/) | Orchestration | `name`+`version` | `dependencies`, `kubeVersion` | Partial (chart repo) | Artefact vs app version split |
| [OpenAPI 3.1](https://spec.openapis.org/oas/v3.1.0) | Orchestration | `info.title`+`info.version` | `servers`, `paths`, SPDX license | Partial | License-identifier alignment with SPDX |
| [Backstage](https://backstage.io/docs/features/software-catalog/descriptor-format/) | DX catalog | kind / namespace / name | `spec.owner`, `providesApis`, `lifecycle` | Yes (Git / LDAP / K8s) | Required-reference ownership |
| [Cortex](https://docs.cortex.io/ingesting-data-into-cortex/entities/yaml) | DX catalog | `x-cortex-tag` (global) | `x-cortex-slos`, custom types | Yes (Git + integrations) | Globally-unique identity |
| [OpsLevel](https://docs.opslevel.com/docs/opslevel-yml) | DX catalog | `name`+`aliases` | `tier`, `lifecycle`, `tools` | Yes (filename + K8s) | Stable aliases for enums |

# Open Questions

- Where does the manifest live? Repo root, served from an endpoint, or both? *Recommendation from synthesis: both — `/.well-known/sff-factory-manifest.json` for discovery and an OCI Artifact for content-addressed retrieval.*
- How is the manifest signed? *Recommendation from synthesis: DSSE envelope (in-toto / Sigstore) anchored on `builder.id`.*
- What is the relationship to SFF-RFC-0019 (OpenLineage Compatibility Layer)? *The `telemetry` block is the bridge.*
- Where does telemetry-emission conformance live — inline (Scorecard-style) or as a referrer-style attestation (SLSA-style)?
- Should conformance claims be self-asserted, signed by the factory, or both?
- Multi-tenant factories — single doc with `components[]` (CycloneDX-style) or many small docs linked by reference (Backstage-style)?

# References

- SPDX Specification 3.0.1. https://spdx.github.io/spdx-spec/v3.0.1/
- CycloneDX Specification Overview. https://cyclonedx.org/specification/overview/
- SLSA Provenance v1.0. https://slsa.dev/spec/v1.0/provenance
- OpenSSF Scorecard. https://github.com/ossf/scorecard
- GUAC. https://guac.sh/
- OCI Image Manifest. https://github.com/opencontainers/image-spec/blob/main/manifest.md
- ORAS / OCI Artifact concept. https://oras.land/docs/concepts/artifact/
- Kubernetes Custom Resource Definitions. https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/
- Helm Charts. https://helm.sh/docs/topics/charts/
- OpenAPI Specification v3.1.0. https://spec.openapis.org/oas/v3.1.0
- Backstage Descriptor Format. https://backstage.io/docs/features/software-catalog/descriptor-format/
- Cortex YAML entity definition. https://docs.cortex.io/ingesting-data-into-cortex/entities/yaml
- OpsLevel Config as Code. https://docs.opslevel.com/docs/opslevel-yml
