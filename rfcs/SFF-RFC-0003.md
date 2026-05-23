---
id: SFF-RFC-0003
title: Agent-Readable Issue Format (ARIF)
state: Discussion
working_group: WG-03
working_group_slug: planning-and-issue-management
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines a structured issue representation that both humans and AI agents can read, write, and decompose. ARIF supersedes the informal Markdown body of typical issues with explicit fields for dependencies, success criteria, agent ownership, and decomposition state.

# Motivation

> TODO: working group to draft.

When agents plan and execute work, the unit of work has to be machine-readable: with explicit dependencies, acceptance criteria, and ownership. Treating issues as unstructured Markdown forces agents to re-derive structure on every read, expensively and inconsistently.

## Related Work

ARIF sits between two traditions: production issue trackers (rich data models, weakly typed for agents) and agent task-decomposition literature / libraries (strongly structured, narrow scope).

**Issue tracker data models** all share an identity / state / body / labels skeleton but diverge sharply on dependency typing and extensibility. [GitHub Issues](https://docs.github.com/en/rest/issues/issues) shipped *typed* `issue_field_values` in 2026 — the first major tracker with first-party custom-field extensibility. [Linear](https://linear.app/developers/graphql) keeps a small workflow `state.type` enum cross-team (`backlog` / `unstarted` / `started` / `completed` / `canceled` / `triage`), which is the cleanest precedent for portable lifecycle reasoning. [Jira](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/) has the widest install base but its `customfield_*` IDs require an out-of-band metadata lookup — agents must learn tenant-specific IDs. [Plane.so](https://developers.plane.so/api-reference/introduction) (AGPL-3.0) exposes *typed custom properties as first-class API objects* — introspectable, unlike Jira's opaque IDs. [OpenProject](https://www.openproject.org/docs/api/endpoints/relations/) (GPL-3.0) has the richest dependency taxonomy in the survey: `relates`, `duplicates`, `blocks`, `precedes` / `follows` (with `lag`), `includes`, `requires`, each carrying a `reverseType`.

**Agent task-decomposition literature and libraries** trade general purpose for explicit structure. [Plan-and-Solve](https://arxiv.org/abs/2305.04091) and [ReAct](https://arxiv.org/abs/2210.03629) produce only free-text or trace-shaped output — low machine-readability. [Tree-of-Thoughts](https://arxiv.org/abs/2305.10601) introduces a per-node LLM evaluator that scores `sure` / `maybe` / `impossible` — a precedent for inline success-likelihood signals. [Hierarchical Task Networks](https://www.sciencedirect.com/topics/computer-science/hierarchical-task-network) (Erol/Hendler/Nau, 1994) are the canonical formal model: `Tasks` (primitive | compound) + `Methods` (decomposition rules with preconditions) + `Operators` (executable actions with pre / effects). [beads](https://github.com/gastownhall/beads) (MIT) — the Foundation's own dependency-aware issue tracker — is the only production library with first-class `acceptance_criteria`, `design` fields, and a `discovered-from` edge that captures agent-time provenance during work. [claude-flow / Ruflo](https://github.com/ruvnet/claude-flow) uses Goal-Oriented Action Planning with explicit `preconditions` + `effects` + `success criteria` per node, generating the DAG from plain-English goals.

The gap ARIF fills sits exactly between these: take a production tracker's pragmatism (extensible, integrable, human-editable) and the literature's rigour (typed dependencies, explicit success criteria, structural decomposition). beads is the closest existing analogue — ARIF generalises it into a portable interchange format.

# Detailed Design

> TODO: working group to draft.

## Proposed core fields (informed by prior art)

**Identity, body, state (tracker-conventional):**

- `id` — stable identifier.
- `title` — single-line summary.
- `state` — small typed enum (Linear-style) for portability: `backlog` | `triage` | `proposed` | `ready` | `in_progress` | `blocked` | `completed` | `cancelled`.
- `description` — human-authored Markdown body.
- `provenance` — `{ created_by, kind: human|agent|system, source_url? }`.

**Success criteria (the lift from beads + GOAP, novel for ARIF):**

- `acceptance_criteria[]` — testable conditions; each entry is `{ id, description, kind: prose|given_when_then|executable_check, evidence? }`. No surveyed tracker has this as a typed field.

**Dependencies (OpenProject vocabulary, normalised):**

- `depends_on[]: ARIFRef[]` — IDs the work depends on (formerly "blocked-by").
- `blocks[]: ARIFRef[]` — typed inverse of `depends_on`.
- `relates_to[]: ARIFRef[]`.
- `duplicates: ARIFRef?` (`duplicated_by` is its symmetric reverse).
- `precedes / follows`, with optional `lag` (OpenProject pattern).
- `discovered_from: ARIFRef?` — agent-time provenance edge (beads pattern).

**Decomposition (HTN-shaped):**

- `kind` — `primitive` | `compound`. Borrowed from HTN.
- `decomposes_into[]: ARIFRef[]` — when `kind == compound`. Distinct from `parent` / `child` so org hierarchy and task expansion aren't overloaded onto one edge.
- `parent: ARIFRef?` — for org / epic hierarchy only.

**Ownership and extensibility:**

- `assignee` — `{ kind: human|agent|unassigned, id? }`.
- `custom_fields[]` — typed (GitHub `issue_field_values`-shaped): `{ name, type: text|single_select|number|date|duration|ref, value }`. Schema-introspectable, not opaque IDs.

## Schema and validation

The ARIF JSON Schema MUST be published alongside this RFC and SHOULD be referenced via `$schema` from every ARIF document.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[GitHub Issues](https://docs.github.com/en/rest/issues/issues) form schemas + `issue_field_values`** — the newest typed-field model in mainstream trackers; ARIF adopts the `{ name, type, value }` shape.
- **[Linear](https://linear.app/developers/graphql)** — strongest GraphQL types and the cleanest small workflow `state.type` enum; ARIF adopts the latter.
- **[Jira Cloud](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/)** — widest install base but opaque `customfield_*` IDs require out-of-band metadata lookup; not portable.
- **[Plane.so](https://developers.plane.so/api-reference/introduction)** (AGPL-3.0) — typed custom properties as first-class API objects; precedent for ARIF's `custom_fields[]`.
- **[OpenProject](https://www.openproject.org/docs/api/endpoints/relations/)** (GPL-3.0) — richest relation taxonomy; ARIF adopts the dependency vocabulary.
- **[beads](https://github.com/gastownhall/beads)** (MIT) — the Foundation's own dependency-aware tracker. Closest existing precedent; ARIF generalises beads' `acceptance_criteria` and `discovered-from` into a portable interchange.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Dependency types | Success criteria field? | Decomposition support | Agent-readable today? | Contribution to ARIF |
|---|---|---|---|---|---|---|
| [GitHub Issues](https://docs.github.com/en/rest/issues/issues) | Tracker | sub-issues, dependencies, parent | No (closed-state reason only) | sub-issues (1 level) | GraphQL + typed `issue_field_values` | Typed-field model |
| [Linear](https://linear.app/developers/graphql) | Tracker | `blocks`, `duplicate`, `related`; parent | No | parent / child + projects | Strong GraphQL types | Workflow `state.type` enum |
| [Jira Cloud](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/) | Tracker | blocks, relates, duplicates, clones (configurable) | No (custom fields possible) | epic→story→subtask | REST / JSON; needs metadata lookup | Cautionary on opaque custom IDs |
| [Plane.so](https://developers.plane.so/api-reference/introduction) | Tracker | parent, links, sub-issue | Via typed custom properties | epics, cycles, modules | OpenAPI + AGPL source | Typed custom props as API objects |
| [OpenProject](https://www.openproject.org/docs/api/endpoints/relations/) | Tracker | relates, blocks, follows (+lag), includes, requires, duplicates | No (custom fields possible) | parent / child + types | HAL+JSON, schema introspection | Dependency vocabulary |
| [Plan-and-Solve](https://arxiv.org/abs/2305.04091) | Lit | Implicit ordering | Implicit (final answer) | Two-phase plan | Free text — Low | Plan-then-execute primitive |
| [ReAct](https://arxiv.org/abs/2210.03629) | Lit | Sequential trace | `Finish[]` action | None explicit | Line-prefixed — Medium | Reasoning + action interleave |
| [Tree of Thoughts](https://arxiv.org/abs/2305.10601) | Lit | Tree edges | Per-node LLM evaluator | BFS / DFS over thoughts | Categorical — Medium | Self-evaluation primitive |
| [HTN (classical)](https://www.sciencedirect.com/topics/computer-science/hierarchical-task-network) | Lit | Ordering, causal, decomposition | Preconditions / effects | First-class, formal | Logic predicates — High | `kind: primitive|compound` |
| [beads](https://github.com/gastownhall/beads) | Lib | `blocks`, `parent-child`, `related`, `discovered-from` | **Yes — `acceptance_criteria`** | Epics, molecules, gates | JSONL + `--json` everywhere | Acceptance criteria, `discovered-from` |
| [claude-flow / Ruflo](https://github.com/ruvnet/claude-flow) | Lib | DAG (pre / effects) | Yes — per-action | GOAP planner | Internal DAG — High | Pre / effects + success criteria |

# Open Questions

- Does ARIF live alongside an existing tracker (GitHub, Linear, Jira) as metadata, or is it standalone? *Recommendation from synthesis: standalone interchange format, with adapters that project to / from popular trackers.*
- How is decomposition tracked across nested agents? *Recommendation: `discovered_from` edge (beads) plus a typed `decomposes_into` distinct from `parent`.*
- Is success criteria a single field, a structured Given/When/Then, or a list of executable assertions (GOAP `effects`)?
- Should ARIF encode workflow `state.type` as a small enum (Linear-style) for portability, or leave state user-defined (Plane / Jira-style)?
- How should agent-time mutations (status, discovery edges) be represented for safe concurrent writes — beads' Dolt + hash-IDs model is the only one in the survey that solves this.

# References

- GitHub Issues REST API. https://docs.github.com/en/rest/issues/issues
- GitHub Issue Fields public preview (2026-03). https://github.blog/changelog/2026-03-12-issue-fields-structured-issue-metadata-is-in-public-preview/
- Linear GraphQL Developer Docs. https://linear.app/developers/graphql
- Jira Cloud REST API v3 — Issues. https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/
- Plane Developer Docs. https://developers.plane.so/api-reference/introduction
- OpenProject API: Relations. https://www.openproject.org/docs/api/endpoints/relations/
- Wang, L. et al. *Plan-and-Solve Prompting.* arXiv:2305.04091. https://arxiv.org/abs/2305.04091
- Yao, S. et al. *ReAct.* arXiv:2210.03629. https://arxiv.org/abs/2210.03629
- Yao, S. et al. *Tree of Thoughts.* arXiv:2305.10601. https://arxiv.org/abs/2305.10601
- Erol, Hendler, Nau. *HTN Planning overview.* https://www.sciencedirect.com/topics/computer-science/hierarchical-task-network
- beads (gastownhall/beads). https://github.com/gastownhall/beads
- claude-flow / Ruflo. https://github.com/ruvnet/claude-flow
