---
id: SFF-RFC-0019
title: OpenLineage Compatibility Layer
state: Pre-Draft
working_group: WG-02
working_group_slug: metadata-storage-for-factories
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines a mapping between Factory Manifest events (SFF-RFC-0002) and the [OpenLineage](https://openlineage.io) specification so factories can publish to existing OpenLineage consumers without parallel instrumentation.

# Motivation

> TODO: working group to draft.

OpenLineage is the de facto data-lineage event spec. Factories that emit Foundation manifest events should be able to also satisfy OpenLineage consumers (Marquez, Datakin, vendor backends) by emitting a compatible projection — without re-instrumenting their pipeline.

## Related Work

**OpenLineage** ([spec](https://github.com/OpenLineage/OpenLineage/blob/main/spec/OpenLineage.md), Apache-2.0, LF AI & Data graduate project) defines a three-entity object model — **Job** `(namespace, name)`, **Run** `(runId UUID)`, **Dataset** `(namespace, name)` — with four event types (`START`, `COMPLETE`, `ABORT`, `FAIL`) plus `RUNNING` and `OTHER` for heartbeats and partial state. Events carry typed [facets](https://openlineage.io/docs/spec/facets/) on job, run, dataset, inputDataset, and outputDataset entities; standard facets include `sourceCodeLocation`, `sql`, `ownership`, `documentation`, `jobType`, `nominalTime`, [`parent`](https://openlineage.io/docs/spec/facets/run-facets/parent_run/) (supports nested run hierarchies via `root`), `schema`, `dataSource`, `lifecycleStateChange`, `columnLineage`, `dataQualityMetrics`. [Naming conventions](https://github.com/OpenLineage/OpenLineage/blob/main/spec/Naming.md) prefix dataset namespaces with the physical scheme (`postgres://`, `s3://`, `bigquery`, `kafka://`); job namespaces derive from the scheduler (`airflow://cluster`). Every event and facet carries `_producer` and `_schemaURL` — the core extensibility hook for vendor-specific facets via the `{prefix}{Name}{Entity}Facet` convention. Transports are pluggable: HTTP (`POST /api/v1/lineage`), Kafka, file, console, no-op, composite. Static-lineage emission (without runs) is an active [proposal (#1837)](https://github.com/OpenLineage/OpenLineage/blob/main/proposals/1837/static_lineage.md).

**Adjacent lineage systems** vary in OpenLineage compatibility. [Marquez](https://marquezproject.ai/) (Apache-2.0, LF AI & Data Sandbox) is the OpenLineage reference consumer — its native model *is* OpenLineage; `POST /api/v1/lineage` is the canonical sink. [Apache Atlas](https://atlas.apache.org/) (Apache-2.0) has a typesystem of `Entity` + `Classification` + `Relationship` with lineage implicit via `Process` entities; Hadoop-era, classification-driven governance, but consumes OpenLineage only via community bridges. [DataHub](https://datahub.com/) (Apache-2.0) uses an entity-aspect graph with URN identities (`Dataset`, `DataJob`, `DataFlow`, `DataProcessInstance`); exposes an OL REST endpoint that translates events into MCPs and has a Spark listener wrapping OL. [OpenMetadata](https://open-metadata.org) (Apache-2.0) is schema-first (JSON-Schema entities with explicit lineage edges); 1.12.x added an OL pipeline connector. [Egeria](https://egeria-project.org) (Apache-2.0, LF AI & Data) ships an OL Integration Connector that ingests OL events into its federated OMRS graph and can re-emit — both producer and consumer.

The gap SFF-RFC-0019 fills: OpenLineage is data-pipeline-centric. Factory events (build / deploy / agent step / approval / gate / cross-agent invocation) are *not* data pipelines. The compatibility layer maps factory concepts to OpenLineage primitives using existing extensibility hooks (custom facets, non-data namespaces), and identifies where SFF must extend OL (gates, approvals, agent identity) without breaking OL consumers.

# Detailed Design

> TODO: working group to draft.

## Mapping outline (informed by deep-read)

| SFF concept | OpenLineage equivalent | Notes |
|---|---|---|
| Factory identity (`factoryId`, version) | `job.namespace = sff://<factory-host>`; `job.name = <factory-id>`; `sourceCodeLocation` facet (git url + sha) | Factory = OL "job definition" |
| Factory run / invocation | `run.runId = <uuid>`; `START` + terminal event | Run = single factory execution |
| Sub-task / agent step | Child `Run` with [`parent` run facet](https://openlineage.io/docs/spec/facets/run-facets/parent_run/) pointing to factory `runId`; recurse for nesting; use `root` for top-of-hierarchy | OL ParentRunFacet handles arbitrary nesting depth |
| Build event (artefact produced) | `outputs[]` Dataset with `namespace=oci://registry`, `name=image@sha`, `lifecycleStateChange=CREATE` + custom `sff_buildBuildJobFacet` (compiler, flags, reproducibility hash) | Treat artefact as dataset |
| Deploy event | New `Run` with `parent` = build run; `outputs[]` = environment dataset (`namespace=k8s://cluster`, `name=ns/deployment`); custom `sff_deployRunFacet` (strategy, revision) | Models deploy as job-on-job |
| Dependency | `inputs[]` Dataset entries (source repo, package, model weights); custom `sff_dependencyDatasetFacet` (resolver, lockfile hash, supply-chain attestation) | Each dep = input dataset |
| Test result | Child `Run` with `jobType=TEST`; custom `sff_testResultRunFacet` (pass / fail / skip counts, coverage); failures → `errorMessage` facet on `FAIL` | Maps cleanly to OL data-quality facets |
| Spec / RFC reference | `documentation` job facet + custom `sff_specRefJobFacet` (rfc-id, version) | Reuses standard facet |
| Agent / model identity | Custom `sff_agentRunFacet` (model, provider, temperature, tokenUsage) | No native OL slot — RFC-novel facet |
| Approval / gate | Custom `sff_approvalRunFacet` (signers, threshold, expiry) | No native OL concept — RFC-novel facet |

## Recommended emission architecture

The factory SHOULD emit **native SFF events** through its own pipeline; a thin **translator service** projects those into OL events on the HTTP transport. Rationale: SFF events will outgrow OL (gates, approvals, multi-agent provenance, cross-org federation). The translator preserves a clean separation. [Marquez](https://marquezproject.ai/), [DataHub](https://datahub.com/), and [Egeria](https://egeria-project.org/) become drop-in consumers.

## Extensibility profile

The factory custom facets follow OpenLineage convention: `{prefix}{Name}{Entity}Facet` where `prefix=sff_`. Each facet declares its own `_schemaURL` hosted under the Foundation's spec URL. Facets MUST be additive — consumers that don't recognise them MUST ignore.

## Non-data namespaces

Container images, binaries, model weights, and merged PRs are mapped to Datasets with non-data namespace schemes:

- `oci://registry/...` — container images
- `git://host/org/repo` — source / merged PRs
- `pypi://`, `npm://`, `crates.io://`, `huggingface://` — package and model references

This is *unusual* for OpenLineage (which assumes data pipelines) but legal — namespace is just a URI scheme. The mapping document SHOULD include a non-data namespace naming-convention appendix and propose it as an addition to [`Naming.md`](https://github.com/OpenLineage/OpenLineage/blob/main/spec/Naming.md) upstream.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **Replace OpenLineage in the SFF ecosystem entirely** (rejected — it already has wide adoption).
- **Native OpenLineage emission only** (rejected — loses factory-specific fields).
- **[Marquez](https://marquezproject.ai/)** (Apache-2.0, LF AI & Data). Reference consumer — useful as the integration target.
- **[Apache Atlas](https://atlas.apache.org/)** (Apache-2.0). Classification-driven governance; OL consumption only via bridges.
- **[DataHub](https://datahub.com/)** (Apache-2.0). Aspect graph; translates OL events into MCPs.
- **[OpenMetadata](https://open-metadata.org)** (Apache-2.0). Schema-first; 1.12.x added OL pipeline connector.
- **[Egeria](https://egeria-project.org)** (Apache-2.0, LF AI & Data). Both producer and consumer via OL Integration Connector.

# Prior Art

> TODO: working group to draft.

| Source | Native model | OL producer? | OL consumer? | Custom facets? | Contribution |
|---|---|---|---|---|---|
| [Marquez](https://marquezproject.ai/) | Namespace / Job / Run / Dataset (OL-shaped) | n/a | Yes (reference) | Yes (stored as JSON) | Canonical sink |
| [Apache Atlas](https://atlas.apache.org/) | Type / Entity + Process | No | Via bridge | Via typedefs | Classification propagation |
| [DataHub](https://datahub.com/) | Entity-aspect graph (URN) | Partial (Spark) | Yes (REST endpoint) | Yes (aspects) | Translates OL → MCP |
| [OpenMetadata](https://open-metadata.org) | JSON-Schema entities | No | Yes (1.12+ connector) | Limited (extension fields) | Schema-first ingestion |
| [Egeria](https://egeria-project.org) | OMRS federated graph | Yes (connector) | Yes (connector) | Yes (OMRS types) | Federation across repos |

# Open Questions

- Are bidirectional consumers (read OpenLineage → produce SFF Factory Manifest events) in scope for v0? *Recommendation from synthesis: out of scope; v0 is producer-side only.*
- Versioning policy — track OpenLineage releases or pin? *Recommendation: pin to a minor version range, document the pin per release.*
- Do we register `sff_*` facets upstream with OpenLineage or keep them vendor-prefixed? *Recommendation: keep vendor-prefixed, contribute promising candidates upstream over time.*
- Does the SFF runId equal the OpenLineage runId, or do we maintain parallel IDs?
- Streaming factories — adopt OL's `RUNNING` heartbeats or rely on `START` / terminal only?
- Naming authority for non-data namespaces (`oci://`, `git://`) — propose to OL's `Naming.md` or document locally?

# References

- OpenLineage. *Specification (OpenLineage.md).* https://github.com/OpenLineage/OpenLineage/blob/main/spec/OpenLineage.md
- OpenLineage. *Naming.md.* https://github.com/OpenLineage/OpenLineage/blob/main/spec/Naming.md
- OpenLineage. *Object Model.* https://openlineage.io/docs/spec/object-model/
- OpenLineage. *Parent Run Facet.* https://openlineage.io/docs/spec/facets/run-facets/parent_run/
- OpenLineage. *Job Hierarchy.* https://openlineage.io/docs/1.40.1/spec/job-hierarchy/
- OpenLineage. *Static Lineage proposal #1837.* https://github.com/OpenLineage/OpenLineage/blob/main/proposals/1837/static_lineage.md
- Marquez Project. https://marquezproject.ai/
- Apache Atlas. https://atlas.apache.org/
- DataHub. *OpenLineage docs.* https://docs.datahub.com/docs/lineage/openlineage
- OpenMetadata. *OpenLineage connector.* https://docs.open-metadata.org/v1.12.x/connectors/pipeline/openlineage
- Egeria. *Lineage management overview.* https://egeria-project.org/features/lineage-management/overview/
