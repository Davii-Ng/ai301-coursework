# Rubric: is this a good first issue?

All recency thresholds are measured against the capture date stated at the
top of the eval bundle (live mode: against today), per
`references/evidence-guide.md`.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| repo-alive | The `archived:` flag on the repo line and the `last 5 default-branch commits` list, both under Repo facts (live mode: the archived banner and the repo front page's commit list). | The repo is NOT archived AND at least one of the last 5 default-branch commits is dated within 60 days of the capture date. An archived repo fails regardless of commit dates. | required |
| maintainer-responds | The `maintainer first-response sample` under Repo facts; the `author_association` of each commenter in this issue's Comments section; the author names and dates of the last 5 default-branch commits. | Passes if ANY of: (a) a commenter in this issue's thread has author_association OWNER, MEMBER or COLLABORATOR; or (b) at least one issue in the first-response sample received a maintainer first response, at any latency; or (c) at least 3 of the last 5 default-branch commits are human-authored (author name does not end in `[bot]`) and dated within 60 days of the capture date — a human merging contributor pull requests counts as maintainer life. Fails only when none of the three holds, i.e. no maintainer has spoken anywhere in the sample or the thread and the commit log is stale or bot-only. | required |
| scope-bounded | The issue body, its labels, its open date, its comment thread, and the `linked PRs:` line under Repo facts. | Fails if ANY of: the issue declares itself a tracking, meta or umbrella issue, or asks for the same change repeated across many files or modules; the thread shows the design is still being debated with no maintainer having settled on one approach, or a maintainer asks a question about what the change should be that the thread never answers; a maintainer states the fix touches core internals; the issue is a pure usage or support question rather than a change; the issue has been open more than 365 days AND has two or more abandoned attempts behind it (closed unmerged linked PRs, or PRs mentioned in the thread that never merged); or the issue asks for a NEW capability or product decision (a new UI surface, a new setting, a new feature) that no maintainer has endorsed, where endorsement means a maintainer opened the issue, a maintainer comment invites or approves the work, or the issue carries a good-first-issue label — this clause also fails when the issue itself flags an unresolved prerequisite (an asset, a spec, or a decision still "TBD"). Otherwise passes. Restoring or fixing existing documented behavior is a bug, not a new capability: the endorsement clause does not apply to it. One reported bug stays one bounded piece of work even when a maintainer enumerates its likely causes or lists optional follow-up ideas inside it — that is a diagnosis, not an umbrella. A terse body, a missing reproduction section, or a bare acceptance checklist does not fail this check — grade the size of the work being asked for, not the polish of the writeup. | required |
| unclaimed | `this issue: assignees:` and `linked PRs:` under Repo facts, plus claim comments and pull-request mentions in the Comments section (where the sidebar and the thread disagree, believe the thread). | Fails if ANY of: an assignee is set; any linked pull request is open; or the thread contains a claim ("I'll take this", "can I work on this", "working on this") or a pull-request mention dated within 365 days of the capture date that no maintainer has since released. Passes when the only linked pull requests are closed and unmerged (abandoned attempts, not claims) and every claim in the thread is older than 365 days. Live mode only: under the Path Review house rule in `scope.md`, other students' claim comments do not block an issue — the assignee and open-linked-pull-request clauses still apply as written. | required |
| ai-policy-allows | The `contribution policy` line under Repo facts (live mode: `CONTRIBUTING.md` in the repo root or `.github/`, the contributor docs it links to, any `AI_POLICY.md` / `AI_USAGE_POLICY.md`, and PR/issue templates). | Fails only on an outright ban on AI-assisted contributions ("we do not accept AI-generated code"). Conditions are not bans: a policy that allows assistive AI use while rejecting fully AI-generated work, or that requires disclosure, personal understanding, testing, or human review, passes. Silence passes: no stated policy, or no CONTRIBUTING.md at all, is not a restriction. | required |

## Verdict rule

Accept only if every required check passes. A single required `fail` produces
`reject`.

`unclear` counts as `fail`: a first issue whose evidence cannot be verified is
not a first issue to take.

The verdict space is binary — `accept` or `reject`. This rubric defines no
`preferred` checks, so nothing modifies a verdict. In live mode, accepted
issues are ranked by the fit profile in `scope.md`; fit never changes a verdict.
