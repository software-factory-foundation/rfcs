---
id: SFF-RFC-0006
title: Universal Test Result Envelope (UTRE)
state: Pre-Draft
working_group: WG-06
working_group_slug: testing-and-test-integrations
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines a standard envelope for test results emitted by factory pipelines: identity, status, evidence, environment, lineage, and severity. UTRE is transport-neutral and accumulates across stages of a single change.

# Motivation

> TODO: working group to draft.

JUnit XML is a quarter-century old and was never designed for AI-augmented pipelines. Modern factories need to attach reasoning, generated test code, screenshots, agent traces, and environment provenance to results — without inventing a new envelope per tool.

## Related Work

The test-result space spans three generations, and no single one carries forward to AI-augmented pipelines unmodified.

**Legacy formats** (1998–2010) — [JUnit XML](https://github.com/testmoapp/junitxml), [TAP 14](https://testanything.org/tap-version-14-specification.html), [xUnit.net XML v2](https://xunit.net/docs/format-xml-v2), [NUnit 3](https://docs.nunit.org/articles/nunit/technical-notes/usage/Test-Result-XML-Format.html), [Maven Surefire](https://maven.apache.org/surefire/maven-surefire-plugin/) — solved CI ingestion universally but lack identity, lineage, and rich evidence. JUnit XML smuggles attachments through `<property>` conventions; only NUnit 3 has a first-class `<attachments>` element. None handles parameterised-test identity or cross-run history.

**Modern formats** (2017–2025) added rich evidence and run history. [Allure](https://allurereport.org/docs/how-it-works-test-result-file/) introduced `historyId` and `testCaseId` for cross-run identity plus behaviour-driven labels (epic / feature / story). [CTRF](https://ctrf.io) (MIT) is the cleanest framework-agnostic single-JSON envelope, with an explicit `environment` block (build / git lineage) and first-class `flaky` / `retries` fields. [OpenTelemetry CI/CD semantic conventions](https://opentelemetry.io/docs/specs/semconv/cicd/cicd-spans/) supply a transport-neutral OTLP shape (`cicd.pipeline.task.run.*`) where multi-stage accumulation falls naturally out of trace/span IDs. [Playwright JSON reporter](https://playwright.dev/docs/test-reporters) and Cypress demonstrate the value of richer artefact types (trace.zip, video) as structured attachments.

**Vendor test schemas** push the state of the art on identity and lineage. [Datadog Test Optimization](https://docs.datadoghq.com/tests/) treats test runs as APM spans (session → module → suite → test) with auto-extracted `git.*` / `ci.*` resource tags and a `test.configuration` discriminator that distinguishes parameterised-test identities. [BuildPulse](https://buildpulse.io) flake-detects on git **tree SHA** — same tree + different result = flaky — needing no source access. [GitHub Step Summaries](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands) is format-agnostic (Markdown via `$GITHUB_STEP_SUMMARY`); third-party renderers (CTRF GitHub Reporter, dorny/test-reporter) parse JUnit / CTRF / TRX. [CircleCI Test Insights](https://circleci.com/docs/collect-test-data/) ingests JUnit XML and joins to workflow / job / parallelism IDs.

The gap UTRE fills sits between CTRF's clean JSON envelope (single-stage) and OpenTelemetry's natural multi-stage accumulation (no rich evidence schema). No surveyed format addresses **multi-stage accumulation across unit → integration → e2e → canary → post-deploy** as a CRDT-friendly append model — UTRE's reason to exist.

# Detailed Design

> TODO: working group to draft.

## Proposed envelope (informed by prior art)

UTRE SHOULD adopt CTRF's outer shape, OpenTelemetry's identity discipline, and Allure / Datadog's history-and-configuration identity.

**Top-level (CTRF-shaped):**

- `schema_version`, `tool` (name / version / instance), `summary` (counts + start / stop), `tests[]`, `environment`, `extra`.

**Identity (OpenTelemetry + Datadog discipline):**

- `change_id` — the change under test (commit SHA, PR, etc.).
- Per test: `id` (UTRE-internal), `history_id` (Allure pattern — stable across runs), `configuration` (Datadog pattern — discriminates parameterised tests), `path` (file / class / method).
- Lineage via OpenTelemetry-style `trace_id` and `parent_span_id` so multi-stage envelopes link.

**Status, severity, evidence:**

- `status` — `pass` | `fail` | `flake` | `skipped` | `error` (CTRF aligned).
- `severity` — `informational` | `minor` | `major` | `blocker`.
- `evidence[]` — typed array with both inline and out-of-band support: `{ kind: screenshot|video|trace|log|coverage|otel_trace|html, uri?, data?: base64, content_type, description? }`. NUnit + Allure + Playwright precedent.

**Environment (CTRF + OTel resource attributes):**

- `environment` — adopts OpenTelemetry `cicd.*` / `vcs.*` resource attribute names so existing collectors light up: `vcs.repository.url`, `vcs.commit.sha`, `vcs.branch`, `cicd.pipeline.name`, `cicd.pipeline.run.id`, `os.name`, `os.version`, `runtime.name`, `runtime.version`.

**Multi-stage accumulation (novel — UTRE's distinct contribution):**

- `stage` — pipeline stage label (`unit` | `integration` | `e2e` | `canary` | `post_deploy` | custom).
- `evidence[]` is append-only per stage; later stages add entries without rewriting prior data.
- `lineage.previous_envelope_id?` — when stages produce distinct envelopes, this links them.
- Quarantine / flakiness state survives across stages and runs (BuildPulse tree-SHA pattern as a referenceable lineage anchor).

## Findings composition

`findings[]` is a structured-assertion array that composes with [SFF-RFC-0008](./SFF-RFC-0008.md) Structured Review Output Schema. UTRE owns identity and lifecycle; SROS owns reviewer-asserted assertions on artefacts.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[JUnit XML](https://github.com/testmoapp/junitxml)** (no license; community schemas Apache-2.0). Universal CI ingestion; cautionary on lineage smuggled through `<property>` conventions.
- **[TAP 14](https://testanything.org/tap-version-14-specification.html)** (Artistic 2.0). Line-stream + YAML diagnostics. Human-readable but no native identity or environment.
- **[NUnit 3](https://docs.nunit.org/articles/nunit/technical-notes/usage/Test-Result-XML-Format.html)** (MIT). First-class `<attachments>` element predates modern formats on this.
- **[Allure](https://allurereport.org/docs/how-it-works-test-result-file/)** (Apache-2.0). `historyId` / `testCaseId` for cross-run identity; behaviour labels.
- **[CTRF](https://ctrf.io)** (MIT). Cleanest framework-agnostic JSON envelope; UTRE adopts its outer shape.
- **[OpenTelemetry CI/CD](https://opentelemetry.io/docs/specs/semconv/cicd/cicd-spans/)** (Apache-2.0). Transport-neutral wire format; multi-stage via trace IDs. UTRE adopts the `vcs.*` / `cicd.*` resource attribute names.
- **[Datadog Test Optimization](https://docs.datadoghq.com/tests/)** (proprietary; client libs Apache-2.0). Best-in-class identity + lineage. `test.configuration` discriminator adopted.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Lineage fields | Attachments | Env capture | Multi-stage accumulation | Contribution to UTRE |
|---|---|---|---|---|---|---|
| [JUnit XML](https://github.com/testmoapp/junitxml) | Legacy | No (via properties) | Via `<property>` convention | Via `<properties>` | No | Universal CI ingestion |
| [TAP 14](https://testanything.org/tap-version-14-specification.html) | Legacy | No | No | No | No | Line-stream + YAML |
| [xUnit.net](https://xunit.net/docs/format-xml-v2) | Legacy | No | No | Limited | No | Test vs env error split |
| [NUnit 3](https://docs.nunit.org/articles/nunit/technical-notes/usage/Test-Result-XML-Format.html) | Legacy | No | First-class `<attachments>` | `<environment>` element | No | Earliest attachments |
| [Allure](https://allurereport.org/docs/how-it-works-test-result-file/) | Modern | `historyId`, `testCaseId` | First-class array | Sidecar `environment.properties` | Via labels / history | Cross-run identity |
| [CTRF](https://ctrf.io) | Modern | `commit`, `branch`, `buildNumber` | First-class array | First-class `environment` | Via `extra` extension | Outer envelope shape |
| [OTel CI/CD](https://opentelemetry.io/docs/specs/semconv/cicd/cicd-spans/) | Modern | `vcs.*`, span / trace IDs | Via span events | OTel resource attrs | Yes (trace IDs) | `vcs.*` / `cicd.*` attribute names |
| [Playwright JSON](https://playwright.dev/docs/test-reporters) | Modern | No | Screenshots / video / trace.zip | `projectName`, tags | No | Trace artefact as attachment |
| [Cypress](https://docs.cypress.io/guides/tooling/reporters) | Modern | No | Video + screenshots | Limited | No | Always-on video |
| [Datadog Test Opt.](https://docs.datadoghq.com/tests/) | Vendor | `git.*`, `ci.*` | Yes | Auto-detected | Yes (session → test) | Most complete identity + lineage |
| [BuildPulse](https://buildpulse.io) | Vendor | Git tree SHA + CI run | Inherits JUnit | Inherits JUnit | Across runs (history) | Tree-SHA flake detection |
| [GH Step Summaries](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands) | Vendor | `GITHUB_*` env vars | Markdown only | Env-derived | No | Format-agnostic surface |
| [CircleCI Insights](https://circleci.com/docs/collect-test-data/) | Vendor | Workflow / job IDs | Inherits JUnit | Inherits JUnit | Across attempts | JUnit-driven |

# Open Questions

- Streaming vs batch — when should a UTRE be considered "final"? *Recommendation from synthesis: per-stage envelopes are independently final; lineage links them.*
- How does UTRE compose with SFF-RFC-0008 (SROS)? *`findings[]` is the bridge; UTRE owns identity, SROS owns assertions.*
- Wire format — JSON envelope (CTRF-like) only, OTLP spans only, or both?
- Quarantine state — boolean (BuildPulse) or workflow-state machine (Datadog)?
- Cross-stage identity — `history_id` per test, per envelope, or both?
- How to surface lineage in formats with no native field (TAP, JUnit) — via a normalisation adapter, or by deprecating ingest?

# References

- testmoapp. *JUnit XML reference.* https://github.com/testmoapp/junitxml
- Test Anything Protocol. *TAP 14 specification.* https://testanything.org/tap-version-14-specification.html
- xUnit.net. *XML v2 format.* https://xunit.net/docs/format-xml-v2
- NUnit. *Test Result XML Format.* https://docs.nunit.org/articles/nunit/technical-notes/usage/Test-Result-XML-Format.html
- Allure. *Test result file format.* https://allurereport.org/docs/how-it-works-test-result-file/
- CTRF (Common Test Report Format). https://ctrf.io
- OpenTelemetry. *CI/CD Spans semantic conventions.* https://opentelemetry.io/docs/specs/semconv/cicd/cicd-spans/
- OpenTelemetry. *CI/CD Resource semantic conventions.* https://opentelemetry.io/docs/specs/semconv/resource/cicd/
- Playwright. *Reporters.* https://playwright.dev/docs/test-reporters
- Cypress. *Reporters.* https://docs.cypress.io/guides/tooling/reporters
- Datadog. *Test Optimization.* https://docs.datadoghq.com/tests/
- BuildPulse. *Flaky tests overview.* https://docs.buildpulse.io/flaky-tests/overview
- GitHub Actions. *Workflow commands.* https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands
- CircleCI. *Collect test data.* https://circleci.com/docs/collect-test-data/
