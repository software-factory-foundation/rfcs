# Contributing to Software Factory Foundation RFCs

This guide describes the lifecycle of an RFC, the voting rules that move it between stages, and the practical Pull Request workflow used in this repository. It mirrors the [Standards Process](https://softwarefactory.org/governance/standards-process) document that the Foundation operates under.

## 1. The RFC Lifecycle

Every Foundation specification advances through five public stages:

### Pre-Draft

A **Pre-Draft** is an initial proposal authored by one or more members and submitted to a Working Group. Pre-Drafts are exploratory, may be revised freely, and carry no normative weight.

→ In this repo: a PR with `state: Pre-Draft` in the front-matter.

### Draft

On Working Group acceptance, a Pre-Draft becomes a **Draft**. Drafts are formally numbered, publicly versioned, and entered into the Foundation's RFC registry (this repo). Drafts may evolve through multiple revisions and undergo Working Group review.

→ In this repo: the PR is merged with `state: Draft`. Subsequent revisions are made via follow-up PRs.

### Last Call

When the Working Group judges a Draft technically complete, it enters a **Last Call** period of not less than thirty (30) days. During Last Call, the wider Foundation membership and the public may submit formal comments. The Working Group must address each comment in the public record.

→ In this repo: front-matter becomes `state: Last Call` with a `last_call_starts` date. Comments are filed as Issues labeled `last-call-comment` and linked to the RFC. The Working Group is required to reply to every comment before advancing.

### Candidate Recommendation

After Last Call, the Draft becomes a **Candidate Recommendation**. A Candidate Recommendation requires at least two independent implementations that demonstrably interoperate against the published conformance criteria.

→ In this repo: `state: Candidate Recommendation`. An `implementations.md` file in the RFC's directory lists the verifying implementations and links to test results.

### Final

On confirmation of conformance and Working Group super-majority vote, the Technical Steering Committee may ratify a Candidate Recommendation as a **Final Standard**. Final Standards are immutable; subsequent changes proceed through a new Draft.

→ In this repo: `state: Final`. The file is moved to `rfcs/final/` and tagged with a git tag `<rfc-id>-final`.

## 2. Voting Rules

The thresholds below are the Foundation's canonical rules; this repository's branch protections and `CODEOWNERS` enforce them.

| Decision | Threshold |
| --- | --- |
| Editorial changes (typos, formatting, link fixes) | Lazy consensus on the public PR. |
| Procedural decisions (timing, scope) | Simple majority of present voting members. |
| Advancement to Last Call | Two-thirds of Working Group voting membership, recorded vote. |
| Ratification to Final | Two-thirds of the Technical Steering Committee, recorded vote, following Working Group recommendation. |

No vote shall be conducted in a private channel. Every advancement vote MUST be recorded as a comment on the corresponding Pull Request (or as a discussion linked from it) within seven (7) days of conclusion.

## 3. Ratification Criteria

For a Candidate Recommendation to advance to Final, the Working Group SHALL publish:

- Conformance criteria sufficient for independent implementers to test against.
- Evidence of at least two independent implementations exercising the full normative scope.
- An interoperability matrix documenting tested behavior.
- Disposition of every Last Call comment.

## 4. Filing a New RFC

1. **Pick a number.** Use the next available `SFF-RFC-XXXX` number. Numbers are reserved when a PR is opened; gaps are filled in order if PRs are abandoned.
2. **Copy the template.** `cp rfcs/TEMPLATE.md rfcs/SFF-RFC-XXXX.md`.
3. **Fill in front-matter.** Set `id`, `title`, `state: Pre-Draft`, `working_group`, `working_group_slug`, `authors`, `created`, `updated`.
4. **Write the body.** Use the Summary / Motivation / Detailed Design / Drawbacks / Alternatives / Open Questions structure. Other sections welcome.
5. **Open the Pull Request.** Title format: `RFC SFF-RFC-XXXX: <title>`. Link the working group in the PR description.
6. **Iterate.** Respond to review comments by pushing additional commits to the PR branch. Don't rebase / squash unless explicitly asked — preserving the discussion trail matters.

## 5. Commenting on an RFC

- **Code-level / wording comments** → use GitHub's PR review feature inline on specific lines.
- **Substantive design comments** → file an Issue with the `rfc-comment` label and reference the RFC ID. The Working Group responds in the issue thread.
- **Last Call comments** → file an Issue with `last-call-comment` label during the Last Call window. These MUST be dispositioned in the public record before ratification.

## 6. Errata and Maintenance

Errata to a Final Standard are issued as numbered amendments (`SFF-RFC-XXXX-Errata-N.md` alongside the final file). Substantive changes that alter normative behavior require a new Draft and SHALL NOT be issued as errata.

## 7. Withdrawal

A Final Standard MAY be withdrawn from active status by a two-thirds vote of the Technical Steering Committee in cases of demonstrated harm, irreparable interoperability failure, or supersession by a successor specification. Withdrawn standards remain published under their original license but are moved to `rfcs/withdrawn/` and tagged `<rfc-id>-withdrawn`.

## 8. IP and Licensing

All contributions are governed by the Foundation's [IP Contribution Policy](https://softwarefactory.org/governance/ip-policy). By opening a Pull Request you:

- Affirm a Developer Certificate of Origin for your contribution.
- Grant the Foundation a perpetual, irrevocable, royalty-free license to publish, sublicense, and incorporate the contribution into Foundation work product.
- Retain copyright and may use your contribution outside the Foundation under any terms you choose.

Voting members are additionally bound by the Patent Non-Assertion Covenant defined in the IP Contribution Policy.

## 9. Antitrust

This repository is a Foundation venue. The [Antitrust & Conflict-of-Interest Policy](https://softwarefactory.org/governance/antitrust-policy) applies to all discussion. Do not raise prices, costs, market shares, customer allocation, or any other commercially sensitive topic in Pull Requests, Issues, or related Discussions.

If a prohibited topic appears, any participant may and should call attention to it. The Working Group chair will end the discussion, record the incident, and seek guidance from Foundation counsel.

## 10. Questions

For procedural questions: file an Issue labeled `process`.
For substantive working group questions: use the [foundation contact form](https://softwarefactory.org/contact?topic=standards).
