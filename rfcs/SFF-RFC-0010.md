---
id: SFF-RFC-0010
title: Factory Evaluation Benchmarks v0
state: Pre-Draft
working_group: WG-10
working_group_slug: continual-improvement
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines a starter set of benchmarks for evaluating software factories on completeness, correctness, speed, cost, and safety. Benchmarks are reproducible, public, and version-pinned so claims of "our factory is better at X" can be compared apples-to-apples.

# Motivation

> TODO: working group to draft.

Every factory vendor publishes self-flattering numbers against their own private rubric. The result is that buyers and members can't compare. A small, public, reproducible benchmark suite — owned by the Foundation rather than any one vendor — gives the field a shared frame of reference.

## Related Work

Factory benchmarking sits at the intersection of code / agent benchmarks, eval methodology, and benchmark governance.

**Code and agent benchmarks** are the closest existing tradition. [SWE-bench](https://www.swebench.com) and [SWE-bench Verified](https://openai.com/index/introducing-swe-bench-verified/) (MIT) use real GitHub issues with hidden test patches as ground truth — but were [deprecated by OpenAI in 2025](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/) citing contamination (post-June-2024 training data exposure) and scaffolding-driven variance, a cautionary tale for static benchmarks. [SWE-bench Live](https://arxiv.org/pdf/2505.23419) continuously sources post-2024 issues to neutralise leakage. [BigCodeBench](https://github.com/bigcode-project/bigcodebench) (Apache-2.0) forces complex multi-library function-call composition (1,140 tasks across 139 libraries, 99% branch coverage). [HumanEval+ / EvalPlus](https://github.com/evalplus/evalplus) (Apache-2.0) catches spec-gaming by expanding the original HumanEval with 80× more LLM+mutation-fuzzed tests. [LiveCodeBench](https://livecodebench.github.io) makes per-problem release-date stamps first-class — time-windowed scoring is built in. [METR's HCAST / RE-Bench](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/) outputs a "time horizon at 50% success" capability number anchored to human-baselined completion time.

**Eval methodology frameworks** define *how* to measure rather than what. [MLPerf](https://github.com/mlcommons/policies) (Apache-2.0 / CC-BY) sets the gold standard for submission integrity: closed/open divisions, peer review of every submission round before publication, frozen reference implementations. [HELM](https://crfm.stanford.edu/helm) (Apache-2.0, Stanford CRFM) treats *holism* as methodology — every model runs on the cross-product of scenarios × metrics (accuracy / calibration / robustness / fairness / bias / toxicity / efficiency), not just accuracy. [BIG-bench](https://github.com/google/BIG-bench) (Apache-2.0) uses canary strings to detect training-data contamination and runs community contribution through peer review. [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) (MIT) — the de-facto open-LLM reference harness — tags every result with task-version integers, seeds, and commit hashes. [OpenAI Evals](https://github.com/openai/evals) and Anthropic's published harnesses contribute model-graded rubrics and dataset-hash pinning.

**Benchmark governance** is the load-bearing layer for foundation-run evals. [MLCommons](https://mlcommons.org/working-groups/) is the closest analogue — a neutral non-profit with working-groups, member orgs, consensus rule-making, round-based release cadences, and cross-vendor peer review. [NIST AI RMF 1.0](https://www.nist.gov/itl/ai-risk-management-framework) and the [NIST-AI-600-1 GenAI Profile](https://airc.nist.gov/airmf-resources/airmf/) define *process governance* (Govern / Map / Measure / Manage) rather than benchmarks themselves. The contamination literature ([survey](https://arxiv.org/html/2406.04244v1), [BetterBench](https://arxiv.org/html/2411.12990v1)) catalogues mitigations: private holdouts, dynamic / temporal benchmarks, canary strings, n-gram overlap checks, refactoring.

The gap SFF-RFC-0010 fills: there is no existing body governing *software-factory* (multi-agent, multi-tool, long-horizon) evaluation. SWE-bench measures isolated PRs, not end-to-end factories; MLPerf measures model serving, not agent workflows. The RFC can carve a niche by combining MLPerf-style submission integrity, lm-eval-harness-style version pinning, HELM's multi-metric matrix, and an MLCommons-style WG charter scoped explicitly to factories.

# Detailed Design

> TODO: working group to draft.

## Proposed benchmark axes (clarified from synthesis)

The five axes — **completeness**, **correctness**, **speed**, **cost**, **safety** — are first-class HELM-style dimensions, evaluated for every task. Each axis carries its own measurement protocol (an early lesson from MLPerf — without protocols, "cost" / "speed" numbers don't compare).

## Submission integrity (MLPerf + lm-eval-harness patterns)

- Every result MUST carry: task version (lm-eval-harness pattern), seed, factory commit / digest, harness commit, model identifiers and versions, hardware accounting (cores / accelerators / wall-clock), and training-data cutoff declaration (contamination-mitigation pattern).
- Every submission round MUST be peer-reviewed before publication (MLPerf pattern).
- Reference implementations are version-pinned at round start and not modifiable except for documented quantization (MLPerf rule).

## Contamination mitigation (synthesis from SWE-bench retirement + BetterBench)

- Public reference set + **mandatory private holdout** with canary strings (BIG-bench pattern).
- Temporal slicing (LiveCodeBench / SWE-bench Live pattern) — tasks tagged with release date so windowed-vs-cutoff scoring is possible.
- Training-cutoff declaration required at submission.

## Multi-metric reporting (HELM pattern)

- Each task produces a vector across the five axes, not a single score. Aggregate scores are derived; raw vectors are authoritative.

## Governance (MLCommons-shaped, novel for factories)

- Foundation WG owns suite definition, round cadence, submission rules, and peer-review process.
- License: Apache-2.0 for both data and harness (HELM / BigCodeBench / MLPerf precedent).
- Suites versioned with date-stamped releases (`SFF-Bench-2026Q3`, ...).

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[SWE-bench / Verified](https://www.swebench.com)** (MIT). Real GitHub issues with hidden test patches. Adopted as the *issue-resolution* slice; cautionary on contamination (OpenAI retirement).
- **[SWE-bench Live](https://arxiv.org/pdf/2505.23419)**. Continuous refresh; adopt the temporal-slicing pattern.
- **[BigCodeBench](https://github.com/bigcode-project/bigcodebench)** (Apache-2.0). Multi-library composition; sandboxed real-time exec.
- **[HumanEval+ / EvalPlus](https://github.com/evalplus/evalplus)** (Apache-2.0). Catches spec gaming; informs the *correctness* axis.
- **[LiveCodeBench](https://livecodebench.github.io)**. Time-windowed scoring is built in.
- **[METR HCAST](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/)**. Time-horizon capability metric — closest precedent for long-horizon factory evaluation.
- **[MLPerf](https://github.com/mlcommons/policies)** (Apache-2.0 / CC-BY). Submission integrity gold standard — adopted.
- **[HELM](https://crfm.stanford.edu/helm)** (Apache-2.0). Multi-metric matrix; adopted as mandatory.
- **[BIG-bench](https://github.com/google/BIG-bench)** (Apache-2.0). Canary-string contamination detection.
- **[lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness)** (MIT). Task-version-integer + seed + commit-hash output convention adopted.
- **[NIST AI RMF + GenAI Profile](https://www.nist.gov/itl/ai-risk-management-framework)**. Process governance reference for the *safety* axis.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Scope | Version-pinned? | Contamination handling | Open submission? | Contribution to v0 |
|---|---|---|---|---|---|---|
| [SWE-bench Verified](https://openai.com/index/introducing-swe-bench-verified/) | Code | 500 curated issues | Yes (frozen) | None native; deprecated by OpenAI | Self-report + leaderboard PR | Issue-resolution slice |
| [SWE-bench Live](https://arxiv.org/pdf/2505.23419) | Code | Rolling post-2024 issues | Snapshot tags | Continuous refresh | Self-report | Temporal slicing |
| [BigCodeBench](https://github.com/bigcode-project/bigcodebench) | Code | 1,140 multi-lib tasks | Yes; HF dataset | Hosted exec sandbox | Leaderboard PR | High test coverage |
| [HumanEval+](https://github.com/evalplus/evalplus) | Code | 164 tasks, 80× tests | Yes | Test expansion | Leaderboard PR | Catches spec gaming |
| [LiveCodeBench](https://livecodebench.github.io) | Code | Dated contest problems | Per-date slices | Date-window scoring | Auto-scraped | Built-in contamination probe |
| [METR HCAST](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/) | Agent | Long agent tasks | Yes | Human-baselined | Limited | Time-horizon metric |
| [MLPerf](https://github.com/mlcommons/policies) | Methodology | HW / SW perf | Round-pinned refs | Peer review of submissions | Yes (structured rounds) | Submission integrity gold standard |
| [HELM](https://crfm.stanford.edu/helm) | Methodology | Scenario × metric grid | Yes; per-run hashes | Public prompts; refresh cycles | Limited (CRFM-run) | Multi-metric mandatory |
| [BIG-bench](https://github.com/google/BIG-bench) | Methodology | 200+ tasks | Per-task | Canary string | Open via PRs | Community contribution + canaries |
| [lm-eval-harness](https://github.com/EleutherAI/lm-evaluation-harness) | Methodology | 200+ tasks | Integer task versions | Per-task notes; commit hash | Open PRs | Output convention |
| [OpenAI / Anthropic evals](https://github.com/openai/evals) | Methodology | Registry-based | Dataset hashes | Private holdouts | Internal | Model-graded rubrics |
| [MLCommons WG](https://mlcommons.org/working-groups/) | Governance | Cross-suite | Round cadence | Submitter peer review | Member-driven | Neutral non-profit template |
| [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework) | Governance | Process | Profile-versioned | N/A | Multi-stakeholder | Process governance |
| [BetterBench](https://arxiv.org/html/2411.12990v1) | Governance | Practices | N/A | Mitigation checklist | N/A | Benchmark hygiene |

# Open Questions

- Should the Foundation host an official leaderboard, or only the benchmark spec? *Recommendation from synthesis: both, MLCommons-style.*
- How are results audited? Self-attestation vs third-party run? *Recommendation: peer-reviewed submissions (MLPerf).*
- Is the unit of evaluation a "factory run" (end-to-end) or a "factory turn" — and how to compare across factories with different decomposition?
- For safety, does the RFC pick its own threats or align with NIST AI RMF GenAI Profile categories?
- Submission license — Apache-2.0 (HELM / BigCodeBench / MLPerf) vs MIT (lm-eval-harness)? *Recommendation: Apache-2.0.*
- How is cost normalised across factories with different cost models (per-token vs subscription vs hosted)?

# References

- Princeton / OpenAI. *SWE-bench Verified.* https://openai.com/index/introducing-swe-bench-verified/
- *Why we no longer evaluate SWE-bench Verified.* https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/
- *SWE-bench Live.* arXiv:2505.23419. https://arxiv.org/pdf/2505.23419
- BigCode. *BigCodeBench.* https://github.com/bigcode-project/bigcodebench
- EvalPlus. *EvalPlus.* https://github.com/evalplus/evalplus
- LiveCodeBench. https://livecodebench.github.io
- METR. *Measuring AI Ability to Complete Long Tasks.* https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/
- MLCommons. *Policies.* https://github.com/mlcommons/policies
- Stanford CRFM. *HELM.* https://crfm.stanford.edu/helm
- Google. *BIG-bench.* https://github.com/google/BIG-bench
- EleutherAI. *lm-evaluation-harness.* https://github.com/EleutherAI/lm-evaluation-harness
- MLCommons. *Working Groups.* https://mlcommons.org/working-groups/
- NIST. *AI Risk Management Framework.* https://www.nist.gov/itl/ai-risk-management-framework
- *Benchmark Data Contamination Survey.* arXiv:2406.04244. https://arxiv.org/html/2406.04244v1
- *BetterBench.* arXiv:2411.12990. https://arxiv.org/html/2411.12990v1
