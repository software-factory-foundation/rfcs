---
id: SFF-RFC-0008
title: Structured Review Output Schema (SROS)
state: Discussion
working_group: WG-08
working_group_slug: code-review-processes
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Specifies the output schema for an automated or human-augmented code review: findings, severity, anchor (file/line/symbol), category, suggested fix, and disposition. Multiple SROS outputs from different reviewers can be merged into a single review.

# Motivation

> TODO: working group to draft.

Code review tooling speaks a hundred different output dialects: GitHub PR comments, SonarQube reports, Semgrep findings, agent-written prose. Merging them into a coherent reviewer experience requires a common schema so the front-end can deduplicate, group by file, and surface the highest-severity findings.

## Related Work

The review-output space has three traditions, and SROS sits at their intersection.

**SARIF and the GitHub Code Scanning ecosystem** define the most complete *single-tool* finding model. The [OASIS SARIF v2.1.0 standard](https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html) provides the canonical anchor (`physicalLocation.region` with start/end line/column/snippet), typed `level` (error/warning/note), `fixes[].artifactChanges[].replacements[]`, `fingerprints` and `partialFingerprints` for stability, `baselineState` (new/unchanged/updated/absent), and `guid`/`correlationGuid` for cross-tool correlation. [GitHub Code Scanning](https://docs.github.com/en/code-security/reference/code-scanning/sarif-files/sarif-support-for-code-scanning) adds a numeric `properties.security-severity` (0.1–10.0) complementing the ordinal `level`, and the [codeql-action fingerprint algorithm](https://github.com/github/codeql-action/blob/main/src/fingerprints.ts) (line-hash of primary location plus context) is the de-facto stability heuristic that lets independent scanners re-upload the same findings idempotently.

**Static-analysis output formats** diverge sharply on suggested-fix shape. [Semgrep](https://github.com/semgrep/semgrep-interfaces/blob/main/semgrep_output_v1.jsonschema) (LGPL-2.1) uses a single string `fix` field — literal range replacement, no AST awareness. [CodeQL](https://docs.github.com/en/code-security/reference/code-scanning/codeql/codeql-cli/sarif-output) emits SARIF with `@kind`, `@problem.severity`, numeric `@security-severity`, and `@precision`. [ESLint](https://eslint.org/docs/latest/use/formatters/) (MIT) has the cleanest multi-candidate model: distinguishes auto-applicable `fix` (range + replacement text) from `suggestions[]` requiring human choice. [Ruff](https://docs.astral.sh/ruff/configuration/) (MIT) gates auto-application with `fix.applicability: safe | unsafe | display`. [Trivy](https://trivy.dev/docs/latest/configuration/reporting/) (Apache-2.0) preserves multiple per-vendor severities (`VendorSeverity`) side-by-side alongside a normalised score — explicit acknowledgement that scorers disagree.

**Review tool data models** model *the conversation*, not just findings. [Gerrit](https://gerrit-review.googlesource.com/Documentation/rest-api-changes.html) (Apache-2.0) is the most complete: `CommentInfo` with `unresolved` flag (the only first-class blocker primitive in the survey), `FixSuggestionInfo` with applyable replacements, and `RobotCommentInfo` as a first-class sibling to human comments. [Phabricator Differential](https://secure.phabricator.com/book/phabricator/article/differential/) introduces draft batching and *ported / ghost comments* that survive diff updates. [Reviewable](https://docs.reviewable.io/reviews.html) makes discussions first-class — they re-anchor to the nearest line across revisions until resolved. [Crucible](https://docs.atlassian.com/fisheye-crucible/3.6.2/wadl/crucible.html) separates `defectRaised` (one reviewer asserted) from `defectApproved` (team agreed). [GitHub PR Review API](https://docs.github.com/en/rest/pulls/reviews) is schema-light: state enum + the ` ```suggestion ` markdown convention; aggregation lives in branch protection rules.

The gaps SROS must fill: **multi-reviewer opinions on the same finding** (only Trivy's `VendorSeverity` and Gerrit's combined-label precedence are precedent), and **anchor drift across diff revisions** (well-solved by Reviewable and Phabricator, absent from SARIF).

# Detailed Design

> TODO: working group to draft.

## Proposed finding shape (informed by prior art)

SROS SHOULD adopt SARIF's spine (anchor + fingerprints + baselineState) and add a thin envelope for multi-reviewer aggregation.

**Identity (SARIF + reviewer attribution):**

- `finding_id` (UUID), `reviewer_id`, `reviewer_kind` — `human` | `llm` | `linter` | `bot`.
- `reviewer_run_id` — the run that produced this finding (SARIF `run.invocations`).
- `correlation_guid` — cross-tool grouping (SARIF pattern).
- `partial_fingerprints` — `{ primary_location_line_hash, ... }` (codeql-action algorithm).

**Anchor (SARIF physicalLocation):**

- `anchor.file` (path).
- `anchor.region` — `{ start_line, start_column?, end_line, end_column?, snippet? }`.
- `anchor.symbol?` — optional logical anchor.

**Category and severity:**

- `category` — `bug` | `smell` | `style` | `security` | `performance` | `doc`. SARIF `properties.tags` is the extension point.
- `severity_ordinal` — `informational` | `minor` | `major` | `blocker` (SARIF-aligned).
- `severity_numeric?` — 0.0–10.0 (GitHub Code Scanning pattern, optional).

**Suggested fix (ESLint + Ruff disambiguation):**

- `suggested_fixes[]` — each entry `{ kind: auto_apply | suggested, applicability: safe | unsafe | display, replacements: [{ region, text }], description? }`.

**Disposition and review-conversation state:**

- `disposition` — `open` | `accepted` | `declined` | `deferred`.
- `state` — `proposed` | `acknowledged` (author saw) | `agreed` (team agreed; Crucible `defectApproved` pattern) | `resolved`.
- `unresolved` — boolean blocker flag (Gerrit pattern).

**Multi-reviewer aggregation (novel — RFC's distinctive contribution):**

- `reviewer_assertions[]` — when multiple reviewers comment on the same finding (matched by fingerprint), each contributes a `{ reviewer_id, severity_ordinal, severity_numeric?, comment }`. The aggregator MUST publish a deterministic merge function (recommended: max severity, with all assertions preserved). Trivy `VendorSeverity` and Gerrit combined-label precedence are the only prior art.

**Anchor drift (Reviewable + Phabricator pattern):**

- `anchor_lineage[]` — `[{ commit_sha, region }]` so a finding can be re-anchored when the diff updates. Combined with `partial_fingerprints` this gives both hash-based stability and explicit re-anchoring.

## Compatibility with SARIF

SROS MUST be losslessly convertible to and from SARIF v2.1.0 for legacy tool interoperation. The conversion drops SROS-specific fields (`reviewer_assertions`, `anchor_lineage`) and maps the rest 1:1.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[SARIF v2.1.0](https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html)** (OASIS standard). Most complete single-tool finding model. SROS adopts its spine and mandates lossless round-trip.
- **[GitHub Code Scanning SARIF](https://docs.github.com/en/code-security/reference/code-scanning/sarif-files/sarif-support-for-code-scanning)**. Numeric severity gating + fingerprint-driven dedup. Adopted as optional numeric severity.
- **[GitHub PR Review API](https://docs.github.com/en/rest/pulls/reviews)**. Schema-light; ` ```suggestion ` markdown convention is universal — SROS suggested-fix output SHOULD render to it.
- **[Gerrit Code-Review](https://gerrit-review.googlesource.com/Documentation/rest-api-changes.html)** (Apache-2.0). `unresolved` flag, `RobotCommentInfo`, applyable `FixSuggestionInfo` — adopted patterns.
- **[Phabricator Differential](https://secure.phabricator.com/book/phabricator/article/differential/)**. Ghost / ported comments — informs the anchor-lineage design.
- **[Reviewable](https://docs.reviewable.io/reviews.html)**. Discussions that re-anchor across revisions — informs anchor-lineage.
- **[Semgrep](https://github.com/semgrep/semgrep-interfaces/blob/main/semgrep_output_v1.jsonschema)** / **[Ruff](https://docs.astral.sh/ruff/configuration/)** / **[ESLint](https://eslint.org/docs/latest/use/formatters/)**. Three patterns for suggested-fix; SROS adopts ESLint's auto/suggested split + Ruff's `applicability` gate.
- **[Trivy](https://trivy.dev/docs/latest/configuration/reporting/)**. `VendorSeverity` — preserves multiple opinions. Informs `reviewer_assertions`.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Anchor precision | Severity model | Suggested fix | Multi-reviewer | Contribution to SROS |
|---|---|---|---|---|---|---|
| [SARIF 2.1.0](https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html) | SARIF | file + region (line+col, start+end) | ordinal `level` | `fix.artifactChanges.replacements` | `correlationGuid` | Spine + fingerprints + baselineState |
| [GH Code Scanning](https://docs.github.com/en/code-security/reference/code-scanning/sarif-files/sarif-support-for-code-scanning) | SARIF | inherits SARIF | ordinal + numeric `security-severity` | yes (via SARIF) | dedup via fingerprints | Numeric severity gating |
| [codeql-action fingerprints](https://github.com/github/codeql-action/blob/main/src/fingerprints.ts) | SARIF | line + N-context hash | n/a | n/a | enables idempotent re-uploads | Reference dedup algorithm |
| [Semgrep JSON](https://github.com/semgrep/semgrep-interfaces/blob/main/semgrep_output_v1.jsonschema) | Static analysis | file + line/col/offset | 5-level | yes — single string replacement | fingerprint only | engine_kind, validation_state extras |
| [CodeQL (SARIF)](https://docs.github.com/en/code-security/reference/code-scanning/codeql/codeql-cli/sarif-output) | Static analysis | file + region | ordinal + numeric + precision | yes (via SARIF) | via SARIF | precision as confidence axis |
| [ESLint JSON](https://eslint.org/docs/latest/use/formatters/) | Static analysis | file + line/col with end | numeric 1/2 | yes — `fix` + `suggestions[]` | none | Cleanest multi-candidate model |
| [Ruff JSON](https://docs.astral.sh/ruff/configuration/) | Static analysis | file + row/col + end | rule-level | yes — `fix.applicability` gate | none | `applicability` enum |
| [Trivy v2](https://trivy.dev/docs/latest/configuration/reporting/) | Static analysis | package + (optional) line | normalised + per-vendor | n/a | `VendorSeverity` | Preserves disagreement |
| [Gerrit](https://gerrit-review.googlesource.com/Documentation/rest-api-changes.html) | Review tool | path + 4-tuple range | label scores; RobotComment.severity | yes — `FixSuggestionInfo` | combined-label precedence | `unresolved`, bot+human unified |
| [Phabricator](https://secure.phabricator.com/book/phabricator/article/differential/) | Review tool | diff + path + line | none explicit | Suggest Edit | blocking reviewers, draft batches | Ported comments across diffs |
| [Reviewable](https://docs.reviewable.io/reviews.html) | Review tool | per-revision per-line, re-anchored | none | via GitHub | per-file LGTM matrix | Discussions survive rebases |
| [Crucible](https://docs.atlassian.com/fisheye-crucible/3.6.2/wadl/crucible.html) | Review tool | file + line; defect flag | `defectRaised` / `defectApproved` | weak | per-reviewer complete | Asserted vs agreed state |
| [GitHub PR](https://docs.github.com/en/rest/pulls/reviews) | Review tool | path + line / start_line + side | review state enum | yes — ` ```suggestion ` | branch protection rules | Schema-light convention |

# Open Questions

- How does SROS compose with SARIF for legacy tools? *Recommendation from synthesis: lossless round-trip with SARIF v2.1.0 is mandatory.*
- What is the minimum interoperability set every implementer MUST support? *Suggested core: identity + anchor + category + severity_ordinal + disposition.*
- Severity vocabulary — normalise to SARIF's 3-level, Semgrep's 5-level, or numeric 0–10? Recommend carry the ordinal as authoritative and the numeric as advisory.
- Is `suggested_fix` a literal text replacement (Semgrep), an AST edit set (ESLint), or a structured `artifactChange` (SARIF)?
- Where does `category` come from — SARIF `properties.tags`, CWE taxonomy, or an SROS-specific enum?
- For multi-reviewer aggregation: max-severity merge, precedence ordering, or quorum-based?

# References

- OASIS. *SARIF v2.1.0 Standard.* https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html
- GitHub. *Code Scanning SARIF support.* https://docs.github.com/en/code-security/reference/code-scanning/sarif-files/sarif-support-for-code-scanning
- GitHub. *codeql-action fingerprints.ts.* https://github.com/github/codeql-action/blob/main/src/fingerprints.ts
- Semgrep. *Output JSON Schema.* https://github.com/semgrep/semgrep-interfaces/blob/main/semgrep_output_v1.jsonschema
- GitHub. *CodeQL CLI SARIF output.* https://docs.github.com/en/code-security/reference/code-scanning/codeql/codeql-cli/sarif-output
- ESLint. *Formatters reference.* https://eslint.org/docs/latest/use/formatters/
- Astral. *Ruff configuration.* https://docs.astral.sh/ruff/configuration/
- Aqua Security. *Trivy reporting.* https://trivy.dev/docs/latest/configuration/reporting/
- Google. *Gerrit changes REST API.* https://gerrit-review.googlesource.com/Documentation/rest-api-changes.html
- Phabricator. *Differential — Inline Comments.* https://secure.phabricator.com/book/phabricator/article/differential_inlines/
- Reviewable. *Reviews docs.* https://docs.reviewable.io/reviews.html
- Atlassian. *Crucible REST API.* https://docs.atlassian.com/fisheye-crucible/3.6.2/wadl/crucible.html
- GitHub. *PR Reviews REST API.* https://docs.github.com/en/rest/pulls/reviews
