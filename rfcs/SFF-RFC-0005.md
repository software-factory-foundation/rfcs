---
id: SFF-RFC-0005
title: Machine-Readable ADR Format (MR-ADR)
state: Discussion
working_group: WG-05
working_group_slug: architecture-and-rules
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Extends the conventional human-readable Architecture Decision Record with strict YAML front-matter and a small grammar for stating the decision, its forces, and its consequences in a form agents can query, link, and check for conflicts.

# Motivation

> TODO: working group to draft.

Classic ADRs are excellent for humans and useless for machines. As agents take on more architectural reasoning, they need ADRs that are queryable: what constraints are in force, what's been deprecated, what trade-offs were accepted.

## Related Work

Architecture Decision Records are well-established as a human practice but only partially structured for machine consumption. Michael Nygard introduced the canonical Markdown template in 2011 ([Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)) with five prose sections (Title, Status, Context, Decision, Consequences) and no front-matter. The de-facto YAML-front-matter successor, [MADR 4.0](https://adr.github.io/madr/) (Markdown Any Decision Records, 2024), formalises `status`, `date`, and RACI-style `decision-makers` / `consulted` / `informed` lists, and is the substrate used by tooling such as [log4brains](https://github.com/thomvaill/log4brains) and the original [adr-tools](https://github.com/npryce/adr-tools). [arc42](https://docs.arc42.org/section-9/) Section 9 reuses Nygard verbatim and contributes the editorial criterion (which decisions warrant an ADR), but no schema.

Two adjacent traditions are relevant. [Y-Statements](https://medium.com/olzzio/y-statements-10eb07b5a177) (Zimmermann, 2012) impose a six-slot grammar — *In the context of X, facing Y, we decided for Z and against W, to achieve Q, accepting R* — forcing capture of rejected alternatives, quality goals, and accepted trade-offs as distinct, slottable units. This is the only existing format whose required slots map cleanly to YAML keys. The [ThoughtWorks Technology Radar](https://www.thoughtworks.com/radar) (CC BY-SA 4.0) demonstrates the opposite end of the spectrum: a flat, fully machine-readable portfolio dataset (ring, quadrant, movement) published as CSV/JSON every edition and rendered by the open-source [BYOR](https://github.com/thoughtworks/build-your-own-radar) tool. It is not an ADR format, but it is the only decision-tracking artefact in widespread use that is queryable as a dataset rather than parsed as prose.

The gap MR-ADR fills is between MADR (front-matter present, but forces and consequences remain prose) and Tech Radar (queryable, but no per-decision granularity). Y-Statements show that the missing slots can be specified without breaking single-file authoring. MR-ADR proposes a strict superset of MADR front-matter that lifts forces, consequences, and cross-references into typed YAML, and commits to a published JSON Schema and a per-repository JSON projection so any agent can validate and query without a custom parser.

# Detailed Design

> TODO: working group to draft.

## Proposed front-matter

The schema is a strict superset of [MADR 4.0](https://adr.github.io/madr/) front-matter — adopting an MR-ADR does not require dropping or renaming existing MADR fields.

**Identity and lifecycle (MADR-compatible):**

- `id` — short ID, e.g. `ADR-0042`. SHOULD be monotonic per repo.
- `title` — single-line noun phrase.
- `status` — enum: `proposed` | `accepted` | `rejected` | `deprecated` | `superseded`.
- `date` — `YYYY-MM-DD` of last status transition.
- `decision_makers[]`, `consulted[]`, `informed[]` — RACI roles (MADR-compatible).
- `tags[]` — free-form labels.

**Structured reasoning (new in MR-ADR):**

- `decision` — single normative statement, the canonical thing the ADR commits to. Mirrors a Y-Statement's "we decided for" slot.
- `forces[]` — each force is an object: `{ type: business|technical|operational|regulatory|other, weight: low|medium|high, description: string }`. Replaces MADR's free-text "Decision Drivers".
- `consequences` — typed object:
  - `positive[]`, `negative[]`, `risk[]`, `neutral[]` — each entry is a `{ description, mitigation?: string }`.
- `rejected_alternatives[]` — each entry: `{ name, why_rejected }`. Lifts Y-Statement's "and against" slot.
- `quality_goals[]` — quality attributes the decision is intended to achieve (e.g., `reliability`, `latency`, `auditability`). Lifts the "to achieve" slot.
- `trade_offs[]` — quality attributes the decision compromises. Lifts the "accepting" slot.

**Typed cross-references (new in MR-ADR):**

- `supersedes: [ADR-NNNN]` — IDs only, not free-text strings.
- `superseded_by: ADR-NNNN`
- `related: [ADR-NNNN]`
- `enables: [ADR-NNNN]`
- `blocks: [ADR-NNNN]`

Cross-references are typed ID arrays rather than the MADR `superseded by ADR-NNNN` string convention, so a validator can detect dangling or cyclic links without string parsing.

## Reference projection

Every repository that adopts MR-ADR SHOULD publish a generated `adrs.json` file (e.g., at the root of an `adr/` directory) that is the concatenation of the front-matter of every ADR, plus a SHA-1 of the body. This makes a repository's decision graph queryable without cloning. The format mirrors [Build Your Own Radar's](https://github.com/thoughtworks/build-your-own-radar) flat-CSV/JSON ethos: one decision = one record.

## Validation

A JSON Schema (`mr-adr.schema.json`) MUST be published alongside this RFC and SHOULD be referenced via a stable URL from each ADR's front-matter (e.g., `$schema: https://softwarefactory.org/schemas/mr-adr/v1.json`).

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **Michael Nygard ADR format** ([Cognitect, 2011](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions), CC0). Prose-only; widely adopted, but forces and consequences remain unstructured text. Insufficient for agent queries.
- **MADR 4.0** ([adr.github.io/madr](https://adr.github.io/madr/), MIT/CC0). Adds YAML front-matter and RACI roles. MR-ADR is a strict superset.
- **arc42 ADR template** ([arc42 §9](https://docs.arc42.org/section-9/), CC BY-SA 4.0). Recommends Nygard verbatim; contributes editorial guidance but no schema.
- **Y-Statements** ([Zimmermann, 2012](https://medium.com/olzzio/y-statements-10eb07b5a177)). Six-slot grammar in a single sentence. MR-ADR adopts the slot structure as YAML keys.
- **ThoughtWorks Technology Radar** ([thoughtworks.com/radar](https://www.thoughtworks.com/radar)). Portfolio-level CSV/JSON dataset. Not per-decision granular, but the queryable-dataset stance informs MR-ADR's `adrs.json` projection.

# Prior Art

> TODO: working group to draft.

The table below summarises the surveyed prior art and where each contributes to MR-ADR.

| Format | Year | Front-matter | Status enum | Supersedes | Queryable | Contribution to MR-ADR |
|---|---|---|---|---|---|---|
| [Nygard ADR](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) | 2011 | No | Free text | String | No | Section conventions, supersession lifecycle |
| [adr-tools](https://github.com/npryce/adr-tools) | 2014 | No | Free text | CLI-mutated string | No | CLI ergonomics; supersession-as-side-effect cautionary |
| [Y-Statements](https://medium.com/olzzio/y-statements-10eb07b5a177) | 2012 | n/a | None | n/a | No (slot-mappable) | `forces`, `decision`, `rejected_alternatives`, `quality_goals`, `trade_offs` slots |
| [arc42 §9](https://docs.arc42.org/section-9/) | 2014 | No | Free text | String | No | Editorial criterion (when an ADR is warranted) |
| [MADR](https://adr.github.io/madr/) | 2017 (v4 2024) | YAML | Enum | String | Partial | Front-matter, RACI roles, status lifecycle |
| [log4brains](https://github.com/thomvaill/log4brains) | 2020 | YAML (MADR) | Enum (MADR) | String (MADR) | Partial | Immutability post-acceptance |
| [TW Tech Radar](https://www.thoughtworks.com/radar) | 2010 | n/a (CSV/JSON) | Ring lifecycle | Edition movement | **Yes** | Queryable-dataset stance; `adrs.json` projection |

Of these, MADR is the most direct technical predecessor and MR-ADR is positioned as a superset of it. Y-Statements are the closest precedent for capturing the structured reasoning slots, and the Technology Radar is the closest precedent for the published-dataset projection.

# Open Questions

- How are MR-ADR conflicts detected at scale?
- Should there be a registry of "Foundation-recognized" constraints?
- Should `forces[]` permit free-form `type` strings, or restrict to the enum above?
- Should the JSON Schema be normative (MUST conform) or informative (validator-driven advisory)?
- Should `adrs.json` be required for conformance, or only recommended?

# References

- Nygard, M. (2011). *Documenting Architecture Decisions.* https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- MADR — Markdown Any Decision Records. https://adr.github.io/madr/ (repo: https://github.com/adr/madr)
- npryce. adr-tools. https://github.com/npryce/adr-tools
- thomvaill. log4brains. https://github.com/thomvaill/log4brains
- arc42 Section 9: Architecture Decisions. https://docs.arc42.org/section-9/
- ThoughtWorks. Technology Radar. https://www.thoughtworks.com/radar (BYOR renderer: https://github.com/thoughtworks/build-your-own-radar)
- Zimmermann, O. (2012). *Y-Statements: A Light Template for Architectural Decision Capturing.* https://medium.com/olzzio/y-statements-10eb07b5a177
- Y-Form Architectural Decision Record. Design Practice Repository. https://socadk.github.io/design-practice-repository/artifact-templates/DPR-ArchitecturalDecisionRecordYForm.html
