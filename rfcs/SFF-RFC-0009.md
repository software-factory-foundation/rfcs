---
id: SFF-RFC-0009
title: Factory Deployment Manifest v0
state: Pre-Draft
working_group: WG-09
working_group_slug: code-deployment
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines the manifest a software factory emits to a deployment system to release a build: identity, environment targets, approval signatures, rollback criteria, and verification probes. Manifests are signed and replayable.

# Motivation

> TODO: working group to draft.

When a factory produces a deployable artifact, the deploy system needs more than the artifact — it needs the policy decisions that authorize the deploy, the environments it's targeted at, and the conditions under which it should be rolled back. A standard manifest lets any factory deploy through any compliant deployer.

## Related Work

Two traditions inform a signed, replayable deployment manifest: software-supply-chain provenance and runtime GitOps specs.

**Supply-chain provenance and attestation** establishes who/what/how an artefact was built. [SLSA Provenance v1.0](https://slsa.dev/spec/v1.0/provenance) (Community Specification License 1.0) wraps a DSSE envelope around an in-toto Statement whose `predicate` carries a `buildDefinition` (with a hard split between *trusted internal* and *untrusted external* parameters) and `runDetails` anchored on `builder.id`. The [in-toto attestation framework](https://github.com/in-toto/attestation) (Apache-2.0) supplies the layered shape (Envelope → Statement → Predicate → Bundle) and the monotonic principle (unrecognised fields MUST be ignored for forward compatibility). [Sigstore](https://docs.sigstore.dev) (Apache-2.0) — cosign + Fulcio (OIDC-bound short-lived certs) + Rekor (transparency log) — enables keyless signing where trust derives from identity + log inclusion rather than a private key. [OCI signing](https://github.com/sigstore/cosign/blob/main/specs/SIGNATURE_SPEC.md) (Apache-2.0) makes signatures *detached* OCI artefacts discoverable by digest. [GitHub Artifact Attestations](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations) is "build provenance as a button" — Sigstore-backed under the hood.

**GitOps and deployment specs** describe runtime intent. [ArgoCD's `Application` CRD](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/) (Apache-2.0) makes the Git revision the desired state; drift is continuously reconciled. [Flux HelmRelease / Kustomization](https://fluxcd.io/flux/components/helm/helmreleases/) (Apache-2.0) introduces *declarative remediation strategies* per phase (`RemediateOnFailure` vs `RetryOnFailure`) and CEL-based custom health gates, and verifies source artefacts via cosign / notation. [OAM](https://oam.dev) (Apache-2.0) cleanly separates developer-defined components from operator-defined traits / policies so apps stay portable; [KubeVela](https://kubevela.io/docs/getting-started/core-concept) implements OAM with workflows as first-class steps (manual approval, multi-cluster topology, rollback). [Argo Rollouts](https://argoproj.github.io/argo-rollouts/features/specification/) (Apache-2.0) treats verification probes as *typed metric queries* (`AnalysisTemplate` with Prometheus / Datadog / web targets) — abort is automatic on failure-budget breach.

The gap SFF-RFC-0009 fills sits between these traditions. Provenance describes build identity; GitOps describes runtime intent. Neither encodes **the deployment decision itself** as a signed, replayable artefact with machine-checkable rollback predicates. The manifest binds them: it is an in-toto Statement *about deploying* (the artefact already has SLSA provenance) and a deterministic specification of *what the runtime must do* (independent of any particular GitOps engine).

# Detailed Design

> TODO: working group to draft.

## Proposed manifest fields (informed by prior art)

The manifest SHOULD be an in-toto Statement with `predicateType=https://sff.dev/deployment-manifest/v0`, wrapped in a DSSE envelope so existing Sigstore / cosign / Rekor verifiers work unchanged.

**Identity and provenance binding:**

- `id` (UUID URN), `version` (SemVer).
- `subject[]` — output artefacts to deploy (digest + name), in-toto pattern.
- `artifact_ref` — primary digest reference; the build's SLSA Provenance attestation is the upstream input.

**Trusted vs untrusted parameters (SLSA split, novel here):**

- `internal_parameters` — factory-set, signed by the factory builder. Verifiers MUST refuse a manifest whose internal-parameters signature doesn't anchor on a recognised `builder.id`.
- `external_parameters` — operator-supplied at approval time, signed at that moment. Permits per-deploy operator overrides without invalidating the factory's signature.

**Target environments and rollout strategy:**

- `target_environments[]` — each `{ name, cluster_ref?, namespace?, strategy: rolling|canary|blue_green, traffic_routing? }`. OAM-style; KubeVela workflow steps inform `strategy`.
- `rollout_steps[]` — optional ordered steps for canary / blue-green with explicit pause / promote / abort gates (Argo Rollouts pattern).

**Approvals (signatures, not workflow steps — novel):**

- `approvals[]` — each approval is an additional DSSE signature on the same Statement (multi-sig over a digest). This makes the manifest replayable without a stateful workflow engine; auditors get one artefact carrying the entire chain of consent.

**Rollback criteria (machine-checkable predicates — novel):**

- `rollback_criteria[]` — typed predicates: `{ probe_ref, condition: failed_for_duration, threshold_seconds, action: revert_to_digest_X }`. Encoded so any compliant deployer behaves identically. Flux's free-form CEL and Argo Rollouts' rollback windows inform but do not constrain the shape.

**Verification probes (Argo Rollouts pattern, typed):**

- `verification_probes[]` — each `{ name, kind: metric|http|analysis_template_ref, target, success_criteria, failure_budget }`. Small typed vocabulary, replayable against historical metric stores.

## Discovery and storage

The manifest SHOULD be published as a sibling OCI artefact (cosign-style detached, addressable by digest) AND optionally pinned to Rekor for transparency-log presence.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[SLSA Provenance v1.0](https://slsa.dev/spec/v1.0/provenance)** (Community Spec License). Provenance is upstream input; SFF-RFC-0009 is the *deployment* statement, distinct predicate type.
- **[in-toto Attestation Framework](https://github.com/in-toto/attestation)** (Apache-2.0). Adopted as the envelope shape verbatim.
- **[Sigstore (cosign + Fulcio + Rekor)](https://docs.sigstore.dev)** (Apache-2.0). Keyless signing path; transparency log presence.
- **[OCI Signature Spec (cosign)](https://github.com/sigstore/cosign/blob/main/specs/SIGNATURE_SPEC.md)**. Detached OCI artefact pattern.
- **[GitHub Artifact Attestations](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations)**. One-button provenance pattern; reference implementation precedent.
- **[ArgoCD Application](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)** (Apache-2.0). Git-revision-as-desired-state — orthogonal layer; the manifest can be consumed by ArgoCD.
- **[Flux HelmRelease](https://fluxcd.io/flux/components/helm/helmreleases/)** (Apache-2.0). Per-phase remediation strategy + cosign source verification.
- **[OAM / KubeVela](https://oam.dev)** (Apache-2.0). Component / trait separation; workflow steps as first-class. The manifest can render to a KubeVela Application.
- **[Argo Rollouts](https://argoproj.github.io/argo-rollouts/features/specification/)** (Apache-2.0). Typed verification probes — adopted as the probe shape.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Signature / attestation | Approval gates | Rollback fields | Verification probes | Contribution to v0 |
|---|---|---|---|---|---|---|
| [SLSA Provenance v1](https://slsa.dev/spec/v1.0/provenance) | Provenance | DSSE (in-toto) | n/a | n/a | n/a | Trusted / untrusted parameter split |
| [in-toto Attestation v1](https://github.com/in-toto/attestation) | Provenance | DSSE envelope | n/a | n/a | n/a | Envelope shape, monotonic principle |
| [Sigstore](https://docs.sigstore.dev) | Provenance | Keyless + transparency log | n/a | n/a | n/a | OIDC-bound certs, Rekor |
| [OCI Signing](https://github.com/sigstore/cosign/blob/main/specs/SIGNATURE_SPEC.md) | Provenance | Detached OCI artefact | n/a | n/a | n/a | Digest-addressable signatures |
| [GH Artifact Attestations](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations) | Provenance | Sigstore-backed | n/a | n/a | n/a | Reference implementation |
| [ArgoCD Application](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/) | GitOps | Git commit signing only | via PR + manual sync | Git revert / `syncPolicy` | `health.lua` checks | Git-revision-as-state |
| [Flux HelmRelease](https://fluxcd.io/flux/components/helm/helmreleases/) | GitOps | cosign / notation on source | `spec.suspend` | `spec.rollback` + remediation | CEL `healthCheckExprs` | Per-phase remediation |
| [OAM](https://oam.dev) | GitOps | Runtime-dependent | `workflow.suspend` | Policy / workflow step | Trait-defined | Component / trait separation |
| [KubeVela](https://kubevela.io/docs/getting-started/core-concept) | GitOps | cosign on addons | `suspend` step | Rollback workflow step | Health traits + workflow | First-class workflow |
| [Argo Rollouts](https://argoproj.github.io/argo-rollouts/features/specification/) | GitOps | Image signing upstream | `pause` step | `rollbackWindow.revisions` | `AnalysisTemplate` (Prom / Datadog) | Typed metric-driven probes |

# Open Questions

- Signature scheme — Sigstore / GPG / custom? *Recommendation from synthesis: DSSE envelope + Sigstore keyless as the recommended path.*
- How does this interoperate with existing GitOps tools (Argo, Flux)? *Recommendation: the manifest is consumed by an adapter that renders to Application / HelmRelease.*
- Do we mint a new predicate type or extend SLSA Provenance with a deployment predicate?
- Where does the manifest live — sibling OCI artefact (cosign-style detached), Rekor entry, or both?
- Multi-environment targets — one manifest per env (Argo Application) or one manifest with `targets[]` (OAM-style)?
- Identity binding — does the deployer need its own OIDC-bound cert for the *act of deploying*, distinct from the build signer?

# References

- SLSA. *Provenance v1.0.* https://slsa.dev/spec/v1.0/provenance
- in-toto. *Attestation Framework.* https://github.com/in-toto/attestation
- Sigstore. *Documentation.* https://docs.sigstore.dev
- Sigstore. *cosign signature spec.* https://github.com/sigstore/cosign/blob/main/specs/SIGNATURE_SPEC.md
- GitHub. *Using artifact attestations.* https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations
- Argo CD. *Declarative Setup.* https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/
- Flux. *HelmRelease component.* https://fluxcd.io/flux/components/helm/helmreleases/
- Open Application Model. https://oam.dev
- KubeVela. *Core Concepts.* https://kubevela.io/docs/getting-started/core-concept
- Argo Rollouts. *Specification.* https://argoproj.github.io/argo-rollouts/features/specification/
